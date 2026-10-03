# Starsector » Trade & Smuggling

[Features](http://fractalsoftworks.com/) [Media](http://fractalsoftworks.com/media) [Blog](http://fractalsoftworks.com/blog) [FAQ](http://fractalsoftworks.com/faq) [Forum](http://fractalsoftworks.com/forum)   [Preorder](https://fractalsoftworks.com/preorder)

## [Permanent Link: Trade & Smuggling](https://fractalsoftworks.com/2014/08/25/trade-smuggling/)

Posted August 25, 2014 by Alex in [Development](https://fractalsoftworks.com/category/development/)

*(If you’ve read an earlier blog post, “[On Trade Design](https://fractalsoftworks.com/2014/03/02/on-trade-design/)“, some of what follows is going to sound familiar.)*

Trade and smuggling are closely related, so it makes sense to tackle both at the same time. Smuggling is simply a more detailed case: trade with complications, if you will.

If you’re going to have a successful trade run of any sort, the first thing you need is information. The main way the player gets information is through news reports and intelligence assessments. Information is important for more than just trade, and these reports have a dedicated tab in the UI.

[Link](https://fractalsoftworks.com/wp-content/uploads/2014/08/intel_report_list.jpg)

For trade, the information the player needs is straightforward: where can they buy or sell something at favorable prices? This kind of information is where things could easily descend into spreadsheet hell, with the player poring over pricing information for every commodity at every market, trying to find the best deals.

It’s important to note that it’s not a binary condition (“too much information” vs “a good amount”); how much information to process is “too much” is subjective. So, the approach to managing the amount of information presented is going to be based largely on my own feelings about what seems right.

Much of the problem is taken care of right off the bat by the economy simulation. When it reaches an equilibrium, prices are such that trade isn’t profitable. For example, if market A produces ore, and market B needs it, the simulation will reach an equilibrium point where the price of ore on both markets is about the same. Throw in tariffs on both ends (set at a brutal 30%), and shipping ore from A to B just isn’t going to bring a profit… unless something happened to disturb the balance.

In some cases, that disruption is directly due to an event. A food shortage will directly increase the price of food. Less obviously, it will also destabilize the local market and decrease the prices of everything else, which may or may not result in other profitable trade runs opening up.

What this means is that you can’t rely on news reports of events being the only way to find out there is a trading opportunity. While the number of these opportunities is much more manageable because they’re mostly driven by events and the simulation actively stamps them out over time, the game still needs to keep track of prices and convey that information to the player.

**Price Updates**

[Link](https://fractalsoftworks.com/wp-content/uploads/2014/08/intel_commodity_prices.jpg)

The idea is to still take advantage of the intel/report system, but add prices as a first-class citizen – something that gets reported on, and something that has a specialized way of being displayed. Price updates come in from all over the Sector, with more updates coming from the star system the player is currently in, and more updates coming from a star system where the player has hacked a comm relay (and thus, presumably, has access to more information).

To keep things manageable, the updates are largely limited to commodities that look “interesting”. Is the price extremely low or high? Interesting. Is the price average, but low compared to *other known prices*? Interesting. Is the commodity something you’ve got a large quantity of in your cargo holds? And so on. In the end, some updates are picked using a weighted random number generator, and the player sees those. (There’s room here for a skill that would increase the amount of information the player gets, and that’s something I’d like to look at in the future. Not looking at skills at all for this update; would be too much to take on all at once.)

In addition, news reports and such can have price updates attached to them. For example, a report about a trade disruption or a food shortage will always give the player new price information for the affected commodities.

Finally, when you trade with a market, you get price updates on commodities with particularly high or low prices.

You might be wondering, “but can’t the player get perfect information about prices by looking at them while trading with a market, information that these price updates don’t provide?” Yes, they can. At first glance, this seems like a problem. We want to avoid a situation where the player can get a leg up by traveling around the Sector and manually noting down all the prices. Fortunately, that’s not very practical due to travel time and expenses. Price information is time-sensitive – you’d be better off taking advantage of an already-known route than spending time trying to find a better one.

[Link](https://fractalsoftworks.com/wp-content/uploads/2014/08/intel_food_prices.jpg)

And so, we have the basis for trade: relevant pricing information being available to the player, in a (hopefully) easy-to-process way.

**Smuggling**Why smuggle rather than trade openly? In game terms, “smuggling” means selling goods on the black market. There are two advantages to this. One, you don’t have to pay the standard 30% tariff. Two, you can also trade in goods that are illegal to buy or sell on the open market, which tend to have higher prices and so frequently have higher profit margins.

Of course, smuggling has its downsides. Trading on the black market is going to damage your reputation with the faction owning the market. Mixing in some open-market trade can cancel this out, as can other reputation-preserving activity such as, say, bounty hunting.

Leaning too heavily on smuggling is going to make it more difficult to carry out – patrols will stop you more frequently, markets will refuse to trade with you (though you’ll be able to trade on the black market still – provided your fleet is small enough, and no patrols are around), and you run the risk of the faction becoming outright hostile. It’s also going to destabilize the markets you’re trading with, particularly if they’re small – leading to lower prices and reduced profits. Or, possibly, increased profits, if the destabilized market is where you’re buying rather than selling.

A particularly profitable form of smuggling is when you’re trading across enemy lines. For example, in the Corvus system, the pirates (or, as they might call themselves, “tax-free operators”) have set up small mining compounds on both moons of the system’s gas giant, *Barad*. Buying ore and volatiles there and selling them elsewhere is very profitable, since there’s no regular trade between pirates and anybody else, and the prices generated by the economic simulation reflect this. Trading with pirates is a sure way to tank your reputation with other factions, though, so the risks rise to match to increased reward.

**Customs Inspections & Tolls**Occasionally, a patrol may decide to stop your fleet and perform a customs inspection. You can try to avoid it – avoiding it is actually easy if your fleet is fast, but avoiding it without a reputation penalty takes a quick response and a fast fleet.

[Link](https://fractalsoftworks.com/wp-content/uploads/2014/08/inspection_comm_ping.jpg)*A short-range comm ping – I wonder what they want? (The answer is: credits. Your credits.)*

The odds of being stopped are much higher if the faction doesn’t like you very much, and as smuggling reduces your relationship with a faction, more frequent customs inspections are a natural consequence.

The patrol performs a cargo scan, which has a chance of detecting any contraband you’re carrying, “contraband” being defined as anything deemed illegal by the patrol’s faction. The patrol also assesses a toll – and a fine, if contraband was found.

[Link](https://fractalsoftworks.com/wp-content/uploads/2014/08/inspection_contraband_found.jpg)*Mr. Petty, customs inspector. I see the random name generator believes form should follow function.*

You gain some reputation by cooperating, and lose some by evading or, worse yet, refusing to comply once contraband is found.

What role does this mechanic play in the larger design, you ask? Good question. A large part of this is flavor – smuggling without the chance of being caught by a patrol just doesn’t feel right. Mechanically, it’s an extra, more interactive risk added to smuggling. Having smuggling be purely penalized by reputation loss felt a bit dry.

It’s also a way to reward higher reputation with a faction. The chances of being stopped drop quickly as reputation levels go up, as do the chances of contraband being found (either the inspectors are “looking the other way”, or they simply aren’t as thorough with someone they trust to some degree).

Finally, it’s a risk added to regular trade, and a way to make smaller ships a little more competitive – an Atlas superfreighter may carry a huge amount of cargo, but it’s not going to be running away from a customs inspector in a frigate any time soon.

**Playstyles**Some playstyles that should be possible as a result of how these mechanics interact:

A small fleet (small enough to sneak into a hostile market – as of this writing, up to 1 destroyer and 1 frigate, or 3 frigates), smuggling to/from pirate-controlled markets. Fast money, cheaper to get started, but a high reputation loss, higher still if evading customs inspections regularly.

A larger fleet, trading mostly in legal goods and smuggling opportunistically. Essentially trading in reputation for money, by occasionally selling on the black market and/or evading authorities, but staying just on the right side of the law in the end.

A larger fleet, engaging in legal trade. Less profit, more tolls, but better relationships all around, and more stable markets – which, in turn, lead to better ships and weapons being available for sale. Gains depend more heavily on taking advantage of events to get higher margins.

Of course, other styles should be possible, too. The above are just the basic ideas, and the details of what kind of playstyle works and how will certainly change due to other mechanics interacting with these.

Comment thread [here](https://fractalsoftworks.com/forum/index.php?topic=8178.0).

Tags: [campaign](https://fractalsoftworks.com/tag/campaign/), [economy](https://fractalsoftworks.com/tag/economy/), [intel](https://fractalsoftworks.com/tag/intel/), [smuggling](https://fractalsoftworks.com/tag/smuggling/), [trade](https://fractalsoftworks.com/tag/trade/)

« [Faction Relationships](https://fractalsoftworks.com/2014/08/12/faction-relationships/)

[Starsector 0.65a Release](https://fractalsoftworks.com/2014/10/20/starsector-0-65a-release/) »

[Link](http://facebook.com/share.php?u=https://fractalsoftworks.com/2014/08/25/trade-smuggling/&t=Trade+%26%23038%3B+Smuggling) [Click to share this post on Twitter](http://twitter.com/home?status=Check%20out%20Starfarer:%20https://fractalsoftworks.com/2014/08/25/trade-smuggling/) [Link](http://reddit.com/submit?url=https://fractalsoftworks.com/2014/08/25/trade-smuggling/&title=Trade+%26%23038%3B+Smuggling) [Link](http://digg.com/submit?phase=2&url=https://fractalsoftworks.com/2014/08/25/trade-smuggling/&title=Trade+%26%23038%3B+Smuggling) [Link](http://stumbleupon.com/submit?url=https://fractalsoftworks.com/2014/08/25/trade-smuggling/&title=Trade+%26%23038%3B+Smuggling)

This entry was posted on Monday, August 25th, 2014 at 7:08 pm and is filed under [Development](https://fractalsoftworks.com/category/development/). You can follow any responses to this entry through the [RSS 2.0](https://fractalsoftworks.com/2014/08/25/trade-smuggling/feed/) feed. Both comments and pings are currently closed.

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