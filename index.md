## NBA DFS

We all know the NBA is the only professional sports league worth paying attention to. We know that! And we also know we can take a few dollars out of the bank and light it on fire at draftkings.com. It’s easy! Kids, [gambling is all around you!](https://www.youtube.com/watch?v=4eHMgXlugIU) 
	
![Image](https://imgur.com/RDZ5f55.jpg)

A few years ago I tried daily fantasy basketball, hoping to turn it into a side hustle to complement poker. I won a little bit, but I was spending hours per slate fine-tuning projections and looking at correlations and spending so much mental energy on it that I was stressed out all the time. I quit after about half a season of playing seriously. 
	
Fast forward to a couple months ago. The NBA, being the only smart and good league, puts their players in a bubble so they can give the fans the playoff basketball they need during COVID-19. I’m learning Python and focusing on data, and I realize I now have the tools I need to automate much of my beloved hobby. 
	
NBA DFS projections from pay sites are pretty decent these days, and since part of my goal is to spend as little time as possible on DFS, I decide to start by averaging projections from the top 3 sites rather than creating my own. It’s kind of annoying to scrape from sites that require credentials, each with different (sometimes difficult) URL paths and weird table formats. All of the sites have one-click .csv downloads, so I find it easier to just bookmark them and download the projections every morning. 
	
Here’s where the fun starts -- my first script has me input the URL of the template for my DraftKings slate and then transforms the projections I downloaded to fit the template. The second script uses the [pydfs-lineup-optimizer](https://github.com/DimaKudosh/pydfs-lineup-optimizer) package to generate the 100 top scoring lineups for the slate. The beauty of using popular pay site projections is that I can  apply linear regression to the projections and lineup %’s to estimate ownership %’s for each player, because everyone else is using the same projections. My ownership projections aren’t bad -- just about as good as the best ones I find online -- and I already know of a couple more predictors that will improve them. 
	
After this all I have to do is bulk upload my lineups to DraftKings and enter contests, right? Well, I could do that, but I wouldn’t win any money. Our goal is to maximize winnings, and in DFS tournaments entering the 100 top expected scoring lineups won’t get you there. I’ll go into detail in my next post.


## NBA DFS - Part 2

The big tournaments (GPPs) on DraftKings are extremely top-heavy. Check out this recent $4 NBA GPP:

Prize pool: $100,000 \
Entries: 29,726 \
Positions paid: 7,513 \
First place: $10,000 \
Second place: $5,000 \
Third place: $3,000 \
Fourth place: $2,000 \
Fifth place: $1,375 \
Seventh place: $1,083.34 \
Tenth place: $1,000 \
52nd: $75 \
101st: $40 \
255th: $20 \
680th: $12.55 \
1,437th: $8.85 \
7,513th: $5.71 \

Rake is a basement bargain at 15.9%. They really love us. The DFS sites have come up with an ingenious prize structure; players can 1-3x their buy-in often enough to keep shoveling money in while still keeping our big score dreams alive.

To 5x your money you need to finish top 0.86%. To 10x, top 0.34%. For a top 10 score (250x+) you have to beat out 99.97% of the field.

![Image](https://media1.tenor.com/images/30767e9d7028583a643eca4f532e6e03/tenor.gif?itemid=4916419)

Consider a roster with 8 players from 8 different teams. The scoring for each player is roughly normally distributed. Since they’re all independent, we just add all the players’ means and variances to get the lineup mean and variance. 

To give ourselves a better chance at tail outcomes, we need to flatten the distribution and/or introduce covariance. There are a few ways to do this:

* Intra-team stacks e.g. Pick and roll combinations, PG and catch-and-shoot 3-point shooter. The point of these is getting lots of points in one real-life event. It’s not as important in NBA as in other sports.
* Game stacks. If there’s a faster than expected pace or OT, your whole lineup benefits.
* Use players whose point distributions inherently have higher variance. There is some science to this; I think I’ll go into detail in a different post.

Additionally, differentiating your lineup by using low-owned players when possible is important, because when your lineup goes off you want to be competing for the top spot against 100 similar lineups rather than 1000. 

You also lower the chance someone duplicates your lineup. Being “duped” is kind of a disaster, all else equal. Imagine a 10-man winner-take-all tournament with 8 unique lineups and 2 identical lineups, all of which have approximately the same point projections*. Each lineup has a 1/9 chance of winning. When the duplicate lineups win, they only win 5 units while everyone else wins 10. Here are the EV’s:

L1: 0.56
L1: 0.56
L2: 1.11
L3: 1.11
L4: 1.11
L5: 1.11
L6: 1.11
L7: 1.11
L8: 1.11
L9: 1.11
------------
Total: 10.0

Notice that even in this dramatic example nobody is winning if rake is 15.6%. I believe the best you can do is 9-10% rake in the $100-$500 buy-in range, and the NFL $3,000 Luxury Box rake is 5.6%. 

I’ll make more DFS posts in the future -- I still want to cover what types of players are high/low variance and share some coding stuff I’m doing.


*In real life, the duplicated lineups will probably have slightly better projections, hence their popularity, but it won’t make up for the dupe effect.


