---
layout: post
title:  "Trilogy of Error (or: Simplicity FTW)"
date:   2015-02-17 00:58:42
categories: 
---

The German speaking movie [Run Lola Run](http://www.imdb.com/title/tt0130827/) was one of my favourite movies in 1998. It tells three variations of the same story with significant different outcome. The makers of [The Simpsons](http://www.imdb.com/title/tt0096697/) were also fascinated by that movie, and created the [Trilogy of Error](http://www.imdb.com/title/tt0701288/) episode as a parody in 2001. And yes, the similarity of the episode name to the Treehouse of Horror episodes is not unintended.

Now we don't have Halloween any time soon, but it's never wrong to share an episode of [Coding Horror](http://blog.codinghorror.com). This is one where I failed three times.

It all started with the intention to make OpenLayers 3 simpler and better by [removing `ol.FeatureOverlay`](https://github.com/openlayers/ol3/pull/3758). A change with 557 lines of code added, and 714 lines removed. The old tests were mostly kept, but no tests were added, and test coverage for the affected components (especially `ol.interaction.Select`) was not good at that time (June 2015). After some discussion, the pull request got merged, and soon we got a [regression](https://github.com/openlayers/ol3/issues/3819) reported, which I [fixed](https://github.com/openlayers/ol3/pull/3820). Side note: I added no new tests with that code.

**Error, episode 1.** The bug reported in [#3878](https://github.com/openlayers/ol3/issues/3878) was about forEachFeatureAtPixel returning the wrong layer if feature already selected. A good bug report, but my fix made `ol.Map#forEachFeatureAtPixel` [ignore layer filters for unmanaged layers](https://github.com/openlayers/ol3/pull/3883). That doesn't sound related, right? In combinagtion with what I had said in [#3820](https://github.com/openlayers/ol3/pull/3820) - "Skipped features need to be hit-detected on unmanaged layers." - the situation now was this: When a feature is selected using `ol.interaction.Select`, it is copied to an unmanaged layer. All good as long as there is only one unmanaged layer which is used by a Select interaction. But what if there are more? Or even multiple Select interactions? Features from that unmanaged layer are hit-detected, and layer filters used by `ol.interaction.Select` are ignored. So each Select interaction can now select features that are not covered by their layer filters!

**Error, episode 2.** There was an unmerged fix for an old issue from the times where we still had `ol.FeatureOverlay`: [Select interaction ignores layer filter for feature overlays](https://github.com/openlayers/ol3/issues/2940). The fix that got finally merged was an updated version of a pull request I had commented with "I really don't understand the implications of this change" - [#4143](https://github.com/openlayers/ol3/pull/4143). Not surprisingly, [another issue](https://github.com/openlayers/ol3/pull/4143#issuecomment-154513975) came up:

> While the old feature overlays couldn't be filtered out, there was a way to effectively ignore them: the callback received a null layer.

Now that sounded like a cure for everything, and I created [#4391](https://github.com/openlayers/ol3/pull/4391) to implement that. But I did not add new tests.

**Error, episode 3.** [#4441](https://github.com/openlayers/ol3/issues/4441) reports an example broken, similar to [#3819](https://github.com/openlayers/ol3/issues/3819), with flickers on hover selection. My fix was [#4450](https://github.com/openlayers/ol3/pull/4450). Very misterious, and also questioned by reviewers. I tried to add missing comments to related tickets, but was not really able to figure out the bigger picture of the misery at that time.

[#4471](https://github.com/openlayers/ol3/issues/4471) reports that a Select interaction applies its select style to other select interactions. A logical consequence of what I said above. And guess what my fix was: [Do not ignore layer filter for unmanaged layers](https://github.com/openlayers/ol3/pull/4472). Duh! What a surprise that reverting the behaviour from my previous fix brought back old issues. Eventually, another regression, [#4822](https://github.com/openlayers/ol3/issues/4822) was reported, bringing back an already fixed problem with two Select interactions on the same layer ([#3878](https://github.com/openlayers/ol3/issues/3878)).

**The solution.** After recapturing what had happened since June 2015, I came to the conclusion that so many regressions with a single component can only be caused by code that is too complicated. Bingo! Why do selected features trigger different hit detection results than features that just sit quitely on a vector layer? They shouldn't! Now I'm hoping that [#4854](https://github.com/openlayers/ol3/pull/4854) - with relevant new tests - puts an end to the never ending feature selection and hit detection story.

Hopefully not to be continued - I'd rather spend my time watching Run Lola Run again :-).


