# Starsector » Once More, with Feeling

[Features](http://fractalsoftworks.com/) [Media](http://fractalsoftworks.com/media) [Blog](http://fractalsoftworks.com/blog) [FAQ](http://fractalsoftworks.com/faq) [Forum](http://fractalsoftworks.com/forum)   [Preorder](https://fractalsoftworks.com/preorder)

## [Permanent Link: Once More, with Feeling](https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/)

Posted October 05, 2018 by Alex in [Development](https://fractalsoftworks.com/category/development/)

If you’ve been keeping up with development progress, you’re probably aware that I’ve been doing some playtesting in preparation for making the next release. Sometimes, playtesting results in relatively small balance tweaks or content and mechanics adjustments. Other times, one finds themselves redoing the economy system for – if I’ve been keeping track correctly – the 5th time. So, that’s what I want to talk about today – briefly! – before diving back into the depths of my TODO list.

The good news is, this iteration of the economy 1) is simple, 2) keeps most of the elements of the [previous one](https://fractalsoftworks.com/2018/01/03/revisiting-the-economy/), 3) is based on colony playtesting, so is likely to actually really stick this time, and 4) as of this writing, fully implemented. This ended up being roughly a week-long detour, by the way, so, all things considered, not too bad.

[Link](https://fractalsoftworks.com/wp-content/uploads/2018/10/food_producers.jpg)

**Why?**
 That is: why rip out the guts of the economy again? It’s about the last thing I wanted or expected to do, but doing actual playtesting with colonies made it clear the previous system just wasn’t good enough. Mainly, it was just too complicated – multiple screens to keep track of colony-to-colony “accessibility” relationships, and then figuring out ways to nudge said relationships a few points to become a supplier and get profits from exports.

This sounded alright on paper, but in practice it was too much to keep track of for something that’s ultimately just manipulating some numbers behind the scenes. Colonies are supposed to be simple to manage, and be the driving force behind other interesting mechanics, not something you spend a lot of time on optimizing. (There were a few other issues, but they mainly stemmed from the same root, and probably aren’t worth talking about. Let’s just say that if *I* find myself being confused by the system, that’s Not Good.)

**New System!**
 And, hopefully, the final one. We’re keeping just about everything – having units of production and demand and so on works nicely and is something that events and player actions (such as piracy or missions) can play into. What’s getting taken out is the system of having colony-to-colony accessibility, and the concomitant approach of finding the best supplier for a commodity.

Instead, each colony has a single accessibility rating. It’s based on its proximity to other colonies, having a spaceport, relevant administrator skills, hostilities with other factions, and so on.

All demand for a commodity across the Sector – let’s say, Food – is added up to produce a “global market value”, that is to say, the amount of profit that’s to be had through Food exports. Likewise, all Food production is totaled up, this time also modified by each producer’s accessibility. A “market share” is calculated for each Food producer, which determines the portion of the global market value it receives as income from exporting that commodity.

Accessibility also limits the amount of a commodity that can be shipped out or brought in, so, for example, an inaccessible market that produces 10 units of Food might only be able to export 3 units, and for anything it needed to import, it’d be limited to only 3 units as well, possibly resulting in shortages.

In-faction accessibility is boosted by a certain amount, meaning that if there’s a high supply of something in the same faction, it’s easier to get a hold of – though this doesn’t generate additional income from exports, but rather represents the ability of a faction to offer not-for-profit logistical support to its colonies.

**Impact**
 This results in a dynamic system where changing hostilities between factions can create shortages and excess stockpiles, giving the player opportunities to exploit.

[Link](https://fractalsoftworks.com/wp-content/uploads/2018/10/supplies_shortages.jpg)

[Link](https://fractalsoftworks.com/wp-content/uploads/2018/10/organics_prices.jpg)

More importantly, however, it has some very nice properties as far as the player’s colonies go.

First off, making some credits on exports is easy. Found a nice planet with Ultrarich Ore Deposits? Get a mining operation going, and you’ll start to make some credits without having to pore through colony-to-colony connections to identify a possible customer. It’s a lot easier to get a slightly-profitable colony off the ground!

Second, producing more of a commodity is always good – it increases your market share and therefore income. There’s no longer such a thing as having too much production.

Third, there are no hard cutoffs (as there were in the previous system) where an extra point of accessibility could mean a huge difference in income, due to being the best supplier for something – or not. It’s also much harder to make completely unreasonable profits by stacking accessibility – instead of becoming the best provider for everything everywhere, you’d “just” have increased market share.

Finally, it’s much easier to get a feel for the state of the Sector and identify opportunities. Consider this screen showing the producers of fuel:

[Link](https://fractalsoftworks.com/wp-content/uploads/2018/10/fuel_producers.jpg)

It’s clear that Sindria is the main producer of Fuel in the Sector, and that there’s not much in the way of other producers. This makes it a market prime for cracking into – almost any amount of Fuel production will get significant market share. And if something unfortunate were to happen to Sindria’s spaceport, nuking its accessibility for a time – or if a raid was to get a hold of the Synchrotron Core powering its production – the potential profits are eye-watering.

Of course, someone at Sindria is probably looking at a similar set of charts on their TriPad and fuming at the market share lost to an irritating upstart…

Comment thread [here](https://fractalsoftworks.com/forum/index.php?topic=13672.0).

Tags: [colony](https://fractalsoftworks.com/tag/colony/), [economy](https://fractalsoftworks.com/tag/economy/), [food](https://fractalsoftworks.com/tag/food/), [fuel](https://fractalsoftworks.com/tag/fuel/), [market share](https://fractalsoftworks.com/tag/market-share/), [market value](https://fractalsoftworks.com/tag/market-value/), [punitive expedition incoming](https://fractalsoftworks.com/tag/punitive-expedition-incoming/)

« [Salvaging Mechanics Update](https://fractalsoftworks.com/2018/09/06/salvaging-mechanics-update/)

[Portrait Hegemonization](https://fractalsoftworks.com/2018/10/16/portrait-hegemonization/) »

[Link](http://facebook.com/share.php?u=https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/&t=Once+More%2C+with+Feeling) [Click to share this post on Twitter](http://twitter.com/home?status=Check%20out%20Starfarer:%20https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/) [Link](http://reddit.com/submit?url=https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/&title=Once+More%2C+with+Feeling) [Link](http://digg.com/submit?phase=2&url=https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/&title=Once+More%2C+with+Feeling) [Link](http://stumbleupon.com/submit?url=https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/&title=Once+More%2C+with+Feeling)

This entry was posted on Friday, October 5th, 2018 at 6:13 pm and is filed under [Development](https://fractalsoftworks.com/category/development/). You can follow any responses to this entry through the [RSS 2.0](https://fractalsoftworks.com/2018/10/05/once-more-with-feeling/feed/) feed. Both comments and pings are currently closed.

[Bluesky](https://bsky.app/profile/amosolov.bsky.social)

[RSS](https://fractalsoftworks.com/feed/)

[Key Recovery](mailto:keys@bmtmicro.com?subject=Starsector%20Activation%20Key%20Recovery&body=%3CPlease%20include%20your%20order%20information%20if%20possible,%20or%20at%20least%20the%20email%20address%20you%20used%20when%20ordering.%3E)

[Contact Us](mailto:fractalsoftworks@gmail.com)

- ## Mailing List
- Item

- ## Categories
  - [Art](https://fractalsoftworks.com/category/art/)
  - [Development](https://fractalsoftworks.com/category/development/)
  - [Lore](https://fractalsoftworks.com/category/lore-2/)
  - [Media](https://fractalsoftworks.com/category/media/)
  - [Modding](https://fractalsoftworks.com/category/modding/)
  - [Releases](https://fractalsoftworks.com/category/releases/)
  - [Uncategorized](https://fractalsoftworks.com/category/uncategorized/)
- ## Archives
  - [June 2026](https://fractalsoftworks.com/2026/06/)
  - [December 2025](https://fractalsoftworks.com/2025/12/)
  - [August 2025](https://fractalsoftworks.com/2025/08/)
  - [July 2025](https://fractalsoftworks.com/2025/07/)
  - [April 2025](https://fractalsoftworks.com/2025/04/)
  - [March 2025](https://fractalsoftworks.com/2025/03/)
  - [December 2024](https://fractalsoftworks.com/2024/12/)
  - [July 2024](https://fractalsoftworks.com/2024/07/)
  - [June 2024](https://fractalsoftworks.com/2024/06/)
  - [May 2024](https://fractalsoftworks.com/2024/05/)
  - [April 2024](https://fractalsoftworks.com/2024/04/)
  - [March 2024](https://fractalsoftworks.com/2024/03/)
  - [February 2024](https://fractalsoftworks.com/2024/02/)
  - [December 2023](https://fractalsoftworks.com/2023/12/)
  - [November 2023](https://fractalsoftworks.com/2023/11/)
  - [October 2023](https://fractalsoftworks.com/2023/10/)
  - [August 2023](https://fractalsoftworks.com/2023/08/)
  - [June 2023](https://fractalsoftworks.com/2023/06/)
  - [May 2023](https://fractalsoftworks.com/2023/05/)
  - [October 2022](https://fractalsoftworks.com/2022/10/)
  - [September 2022](https://fractalsoftworks.com/2022/09/)
  - [July 2022](https://fractalsoftworks.com/2022/07/)
  - [April 2022](https://fractalsoftworks.com/2022/04/)
  - [March 2022](https://fractalsoftworks.com/2022/03/)
  - [January 2022](https://fractalsoftworks.com/2022/01/)
  - [December 2021](https://fractalsoftworks.com/2021/12/)
  - [September 2021](https://fractalsoftworks.com/2021/09/)
  - [July 2021](https://fractalsoftworks.com/2021/07/)
  - [May 2021](https://fractalsoftworks.com/2021/05/)
  - [March 2021](https://fractalsoftworks.com/2021/03/)
  - [December 2020](https://fractalsoftworks.com/2020/12/)
  - [August 2020](https://fractalsoftworks.com/2020/08/)
  - [April 2020](https://fractalsoftworks.com/2020/04/)
  - [February 2020](https://fractalsoftworks.com/2020/02/)
  - [November 2019](https://fractalsoftworks.com/2019/11/)
  - [July 2019](https://fractalsoftworks.com/2019/07/)
  - [May 2019](https://fractalsoftworks.com/2019/05/)
  - [November 2018](https://fractalsoftworks.com/2018/11/)
  - [October 2018](https://fractalsoftworks.com/2018/10/)
  - [September 2018](https://fractalsoftworks.com/2018/09/)
  - [August 2018](https://fractalsoftworks.com/2018/08/)
  - [June 2018](https://fractalsoftworks.com/2018/06/)
  - [May 2018](https://fractalsoftworks.com/2018/05/)
  - [March 2018](https://fractalsoftworks.com/2018/03/)
  - [February 2018](https://fractalsoftworks.com/2018/02/)
  - [January 2018](https://fractalsoftworks.com/2018/01/)
  - [December 2017](https://fractalsoftworks.com/2017/12/)
  - [November 2017](https://fractalsoftworks.com/2017/11/)
  - [September 2017](https://fractalsoftworks.com/2017/09/)
  - [August 2017](https://fractalsoftworks.com/2017/08/)
  - [July 2017](https://fractalsoftworks.com/2017/07/)
  - [June 2017](https://fractalsoftworks.com/2017/06/)
  - [April 2017](https://fractalsoftworks.com/2017/04/)
  - [March 2017](https://fractalsoftworks.com/2017/03/)
  - [February 2017](https://fractalsoftworks.com/2017/02/)
  - [January 2017](https://fractalsoftworks.com/2017/01/)
  - [December 2016](https://fractalsoftworks.com/2016/12/)
  - [November 2016](https://fractalsoftworks.com/2016/11/)
  - [September 2016](https://fractalsoftworks.com/2016/09/)
  - [August 2016](https://fractalsoftworks.com/2016/08/)
  - [July 2016](https://fractalsoftworks.com/2016/07/)
  - [June 2016](https://fractalsoftworks.com/2016/06/)
  - [May 2016](https://fractalsoftworks.com/2016/05/)
  - [April 2016](https://fractalsoftworks.com/2016/04/)
  - [February 2016](https://fractalsoftworks.com/2016/02/)
  - [January 2016](https://fractalsoftworks.com/2016/01/)
  - [December 2015](https://fractalsoftworks.com/2015/12/)
  - [November 2015](https://fractalsoftworks.com/2015/11/)
  - [October 2015](https://fractalsoftworks.com/2015/10/)
  - [September 2015](https://fractalsoftworks.com/2015/09/)
  - [August 2015](https://fractalsoftworks.com/2015/08/)
  - [July 2015](https://fractalsoftworks.com/2015/07/)
  - [May 2015](https://fractalsoftworks.com/2015/05/)
  - [April 2015](https://fractalsoftworks.com/2015/04/)
  - [March 2015](https://fractalsoftworks.com/2015/03/)
  - [February 2015](https://fractalsoftworks.com/2015/02/)
  - [November 2014](https://fractalsoftworks.com/2014/11/)
  - [October 2014](https://fractalsoftworks.com/2014/10/)
  - [August 2014](https://fractalsoftworks.com/2014/08/)
  - [July 2014](https://fractalsoftworks.com/2014/07/)
  - [June 2014](https://fractalsoftworks.com/2014/06/)
  - [May 2014](https://fractalsoftworks.com/2014/05/)
  - [April 2014](https://fractalsoftworks.com/2014/04/)
  - [March 2014](https://fractalsoftworks.com/2014/03/)
  - [January 2014](https://fractalsoftworks.com/2014/01/)
  - [December 2013](https://fractalsoftworks.com/2013/12/)
  - [October 2013](https://fractalsoftworks.com/2013/10/)
  - [September 2013](https://fractalsoftworks.com/2013/09/)
  - [August 2013](https://fractalsoftworks.com/2013/08/)
  - [July 2013](https://fractalsoftworks.com/2013/07/)
  - [June 2013](https://fractalsoftworks.com/2013/06/)
  - [May 2013](https://fractalsoftworks.com/2013/05/)
  - [April 2013](https://fractalsoftworks.com/2013/04/)
  - [March 2013](https://fractalsoftworks.com/2013/03/)
  - [February 2013](https://fractalsoftworks.com/2013/02/)
  - [January 2013](https://fractalsoftworks.com/2013/01/)
  - [November 2012](https://fractalsoftworks.com/2012/11/)
  - [October 2012](https://fractalsoftworks.com/2012/10/)
  - [September 2012](https://fractalsoftworks.com/2012/09/)
  - [August 2012](https://fractalsoftworks.com/2012/08/)
  - [July 2012](https://fractalsoftworks.com/2012/07/)
  - [June 2012](https://fractalsoftworks.com/2012/06/)
  - [May 2012](https://fractalsoftworks.com/2012/05/)
  - [April 2012](https://fractalsoftworks.com/2012/04/)
  - [March 2012](https://fractalsoftworks.com/2012/03/)
  - [February 2012](https://fractalsoftworks.com/2012/02/)
  - [January 2012](https://fractalsoftworks.com/2012/01/)
  - [December 2011](https://fractalsoftworks.com/2011/12/)
  - [November 2011](https://fractalsoftworks.com/2011/11/)
  - [October 2011](https://fractalsoftworks.com/2011/10/)
  - [September 2011](https://fractalsoftworks.com/2011/09/)
  - [August 2011](https://fractalsoftworks.com/2011/08/)
  - [July 2011](https://fractalsoftworks.com/2011/07/)
  - [June 2011](https://fractalsoftworks.com/2011/06/)
  - [May 2011](https://fractalsoftworks.com/2011/05/)
  - [April 2011](https://fractalsoftworks.com/2011/04/)
  - [March 2011](https://fractalsoftworks.com/2011/03/)
  - [February 2011](https://fractalsoftworks.com/2011/02/)
  - [January 2011](https://fractalsoftworks.com/2011/01/)
  - [December 2010](https://fractalsoftworks.com/2010/12/)
  - [November 2010](https://fractalsoftworks.com/2010/11/)
- ## Other - [Log In](http://www.fractalsoftworks.com/wp-admin)

![Blog](https://fractalsoftworks.com/wp-content/themes/starfarer/images/button_blog1.png) ![Media](https://fractalsoftworks.com/wp-content/themes/starfarer/images/button_media1.png) ![FAQ](https://fractalsoftworks.com/wp-content/themes/starfarer/images/button_faq1.png) ![Features](https://fractalsoftworks.com/wp-content/themes/starfarer/images/button_features1.png) ![Digg it!](https://fractalsoftworks.com/wp-content/themes/starfarer/images/social/digg_26_over.png) ![Share this on Facebook](https://fractalsoftworks.com/wp-content/themes/starfarer/images/social/facebook_26_over.png) ![Reddit](https://fractalsoftworks.com/wp-content/themes/starfarer/images/social/reddit_26_over.png) ![Stumbleupon it!](https://fractalsoftworks.com/wp-content/themes/starfarer/images/social/stumbleupon_26_over.png) ![Tweet it!](https://fractalsoftworks.com/wp-content/themes/starfarer/images/social/twitter_26_over.png) ![Preorder now!](https://fractalsoftworks.com/wp-content/themes/starfarer/images/preorder_now1.png) ![Preorder](https://fractalsoftworks.com/wp-content/themes/starfarer/images/button_preorder1.png)