## Betting Totals - 4/21/2021

When you bet sports the book takes a vig. Usually the bettor has to lay -110 (betting 11 to win 10), which equates to a 52.38% win probability needed to break even. I found `spread` -> `moneyline` conversions with a Google search, but I didn’t see any public tables for `total` -> `win probability`.

It’s easy to calculate for any sport if we can assume that...

1. The total (total points scored in each game) is normally distributed
2. The book’s total is close to the mean score of the game (i.e. the book’s number is sharp)

### Step 1: Get game results

I wrote a Python script to download all NBA games from the ‘19-’20 and ‘20-’21 seasons on basketball-reference.com, almost 2000 games as of April 2021. Alternatively, you can just copy/paste the tables into a spreadsheet.

### Step 2: Examine the data

Make a histogram of the total points. If it looks like the IQ meme you’re good to go. If it doesn’t, then you don’t have a normal distribution. 

![image](https://imgur.com/HAnUdDo.jpg)

![image](https://imgur.com/AtgxkL3.jpg)

Nice.

If you don’t want to figure out what kind of distribution you have, you can still get a pretty accurate result as long as you have enough games. Just do a `=COUNTIF(totals, “<x”)` for every x around the mean and then move to step 4.

### Step 3: Calculate the mean and standard deviation

Use the builtin spreadsheet functions to do this. For NBA games the past 2 seasons, we have…

Mean: 223.43
Standard deviation: 20.11

### Step 4: Calculate the P(score < x) for numbers close to the mean

This is just calculating the area under the curve at x. You can use the builtin NORMINV function for this. For NBA, the probability that the total will be under 220 for a game with a 223.43 mean score is 43.23%.  

![image](https://imgur.com/hXmkBBh.jpg)

### Step 5: Calculate the change in win percentage for each point of total

This is the delta column above. If we assume the market total is close to the mean score of the game, we can just use the deltas close to the mean since that’s where we’ll always be betting.


Each point of the total is worth about 2% win probability. We need 2.38% for a profitable bet at -110, so we need a 1.5 point edge on the total to place a +EV bet.

Happy punting 🏈




## NBA data and 2-pointer analysis - 3/3/2021

I recently got my little NBA play-by-play data pipeline operational. It spits out some cool stats that you can’t find on free sites, and today I want to take a look at the leaders for various types of 2pt field goals this season. 

#### Skip to part 2 if you don't care about the programming part.

### Pt. 1 - Small data engineering

I have Python scripts to…
* Update my list of player-team pairs that played the current season (to account for trades and G-league contracts)
* Scrape play-by-play data from basketball-reference.com into a database
* Parse the play columns into a separate database in a query friendly format
* Generate a text summary of last night’s games

Then every day at 8am cron runs a shell script that runs the Python scripts in sequential order:

```shell
#!usr/bin/bash

# activate venv
source ~/.virtualenvs/nbaenv/bin/activate

scripts=(~/NBAdraft/save_to_db/get_players.py \
~/NBAdraft/save_to_db/pbp_yesterday.py \
~/NBAdraft/save_to_db/parse_plays.py \
~/NBAdraft/save_to_db/summary.py)

exit_status=$?
date=$(date)

# run scripts sequentially, exiting if one fails
for s in ${scripts[*]}; do
        python3 $s
        if [ "${exit_status}" -ne 0 ]; then
                ( echo "$date -- $s failed"; echo "" ) >> ~/tmp/nba_status.txt
                echo "exit ${exit_status}"
        fi
done

# copy parsed db to windows desktop
cp NBAdraft/save_to_db/nba_parsed.db /mnt/c/Users/john/Desktop

( echo "$date -- script successful"; echo "" ) >> ~/NBAdraft/save_to_db/daily-cron-status.txt
```

The plays come in sentences like
> M. Wagner makes 2-pt jump shot from 9 ft (assist by R. Westbrook)

so we have to split these into columns that make sense, then replace the first initial with the full name so we don’t accidentally pad the stats of the Curry’s and Ball’s of the world. Here are the new columns:

![image](https://imgur.com/fqK6Yre.jpg)

And this is what the text summary looks like: 
![image](https://imgur.com/RVWim9t.jpg)

So yeah, every day I wake up and all this stuff has already happened. Pretty cool; all the code is on github. In the future I’ll add some other tables to this database like box stats, advanced stats, physical stats, college, etc, which will enable some fun joins. 


### Part 2 - NBA 2-pt leaders

Last night I watched the Phoenix Suns take care of the AD-less Lakers. I drastically underrated them pre-season, despite being right about most of the individual advanced metrics (minus Mikal Bridges who became probably the best 3 and D wing in the league this year). 

Advanced stats don’t love Devin Booker. But when you watch the Suns you realize their plan is to hunt for good shots, then put the ball in his hands with 6 seconds left if they don’t find anything. He gets left holding the entire bag of tough 2’s because he’s a walking bucket. Nice system for the team’s offense, which is currently 8th in efficiency, but bad for Booker’s personal advanced metrics.

So while long 2’s are the worst shot in basketball, they have their place. It’s not as simple as “long 2’s lol” (unless your name is Russell Westbrook). With that let’s get to the good stuff.

Dunks and dunk assists \
![image](https://imgur.com/UNt49uX.jpg)

![image](https://imgur.com/8tdsv73.jpg)

Dunks are good. Guys like Rudy, Mitch Rob, and Thomas Bryant are always going to be near the top in eFG because many of their shots are dunks worth 1.8 points per attempt. 

Props
* The Showtime Golden State Warriors have a rookie in the top 10 (Wiseman) and the only non-big-man in the top 20 (Oubre), both feasting on the delicacies Draymond and Steph are serving up
* Our Time Lord and Savior Robert Williams III cracks the top 20, inexcusably playing only 16 minutes behind the corpse of Tristan Thompson


Slops
* DeAndre Ayton plays 32 minutes a night with Chris Paul, Devin Booker, and two good corner 3 threats, and he isn’t top 20 in dunks or layups. Absurd. He usually rolls to the free throw line, stops, and puts his hand up like he wants to be fed.

Layups \
![image](https://imgur.com/hm59xch.jpg)

Layups are the other desirable type of 2-pointer. Guys miss a lot of layups, but if you can finish low-mid 50%s or higher, you’re doing fine work from an efficiency standpoint. Lots of fouls come on layup attempts as well.

Props
* Zion is doing peak Shaq things around the rim only 57 games into his career. He leads the league in shots at the rim by a mile (24 vs. peak Shaq’s 26), and his TS% is already better than peak Shaq (64% vs. 58%) in part due to his solid free throw shooting
* Bradley Beal showing why he’s led the league in scoring wire to wire so far. We know he can shoot, but he’s also the best guard at both getting to the rim and finishing this year

Slops
* Andre Drummond has always had piss poor efficiency for a big man, but he’s found a new gear this year. He’s taking more shots than ever and achieving career low efficiency

Here are the guys who take the most dunks + layups and their respective rim field goal %s

![image](https://imgur.com/juUdS6h.jpg)

Hook shots \
![image](https://imgur.com/vceLDwE.jpg)

I don’t really know what to think about hook shots. They are almost as prevalent as dunks, but the large variation in fg% suggests some guys are good at them and some are not. Sabonis is a footwork king around the basket and his layup fg% reflects that, but his hook game looks dreadful this year. I’ll take a look at his hook shots from years past another time.



Props
* Vucevic. The volume and consistency are there. He’s not playing with all-stars so it’s much tougher for him to get uncontested layups and dunks than the average big man. Nice offensive player

Slops
* Andre Drummond and Tristan Thompson, as always


Short-mid 2’s (under 14ft, not counting hook shots, layups, or dunks) \

![image](https://imgur.com/qRq83Zq.jpg)

This category includes floaters and short jump shots. I wish I could separate out floaters because they’re a huge chunk of these short-mid 2’s. You must be hitting these at 50-55%+ for them to be a big part of your game.

Props
* Guys whose names start with K

Slops
* Poor DeMar DeRozan held Toronto back for years by using possessions on his 44% midrange jumper
* Trae takes a lot of floaters off the pick and roll, and it does not look like they’re going in enough. His 2P% is usually sub-50% overall, including layups. I’d like to see Trae pass on more of these and go to Gallinari or Collins late in the shot clock. He’s probably just too small to ever be a bigtime late shot clock guy inside the 3pt line.

Long 2’s (over 14ft) \

![image](https://imgur.com/7R60f2a.jpg)

The dreaded long 2, the worst theoretical shot in basketball. Even the most talented bucket-getters usually can’t do better than 1.1 points per attempt on these, which is why they should be avoided until late in the shot clock.

Props
* I’m gonna throw Bradley Beal another prop here. What is he supposed to do, pass to Russ?
* Jokic is just an alien. Nobody should have 65% true shooting while being top 10 in hook shots and top 15 in long 2’s.

Slops
* Jayson Tatum’s shot selection is worse than ever, and he’s not hitting. This is blasphemy, but it looks like Brad needs to get his head out of his ass with this team’s offense. I would like to see them play a Utah Jazz offense with Kemba, Smart, Brown, Tatum, Williams. If they find that, I think they will be a problem in the playoffs


So that’s it. This has been fun to write -- please tell me what you think on Twitter!

\

## Get in loser we're botting - 10/10/20

![Image](https://i.kym-cdn.com/entries/icons/facebook/000/014/008/o-TINA-FEY-BIRTHDAY-570.jpg)

Today we’re making a bot that scrapes NFL game lines from DraftKings. You can find all the code [here.](https://github.com/jmoore87jr/sportsbook-scrapers) 

To get the lines, we’re going to use Python packages ‘requests’ and ‘BeautifulSoup’ (BS). Requests will let us access the site and then BS will parse its html into a soup object. Everything BS does interacts with this soup object.

```python
# create soup
r = requests.get("https://sportsbook.draftkings.com/leagues/football/3?category=game-lines&subcategory=game")
src = r.content 
soup = BeautifulSoup(src, 'lxml')
```
Note: lxml is a parser you can get with ‘pip3 install lxml’, but you can also leave it out.

Now, to get the information we need, we have to inspect the html of the DraftKings page. You can right click anywhere on the page and click ‘Inspect’. It will look like this: 

![Image](https://i.imgur.com/QXtHukO.png)

Each day of games is stored in a separate table. Reading the html, we see that every table sits in a ```<div class=’sportsbook-table__body’> </div>``` wrapper. Here’s how to get a list of each table as a soup object: 

```python
tables = soup.findAll('div', {'class': 'sportsbook-table__body'})
```
Reading the html some more, we see that each of the 4 columns (teams, spreads, totals, moneylines) were children of the parent tables. I split the tables into columns and then extracted the text I needed from each:

```python
# team names
team_names = [ node.text for node in teams.findAll('span', {'class': 'event-cell__name'}) ]
# spreads
spread_nums = [ node.text for node in spreads.findAll('span', {'class': 'sportsbook-outcome-cell__line'}) ]
# spread price
spread_prices = [ node.text for node in spreads.findAll('span', {'class': 'sportsbook-odds american default-color'}) ]
# totals
total_nums = [ node.text for node in totals.findAll('span', {'class': 'sportsbook-outcome-cell__line'}) ]
# totals price
total_prices = [ node.text for node in totals.findAll('span', {'class': 'sportsbook-odds american default-color'}) ]
# moneyline
moneyline = [ node.text for node in moneylines.findAll('span', {'class': 'sportsbook-odds american default-color'}) ]
```
Based on what I’ve read, BeautifulSoup is by far the friendliest web scraping tool out there for Python. Watch a couple of YouTube tutorials, read the [docs](https://www.crummy.com/software/BeautifulSoup/bs4/doc/), and try it out. 

Then we simply turn our 6 lists into a pandas DataFrame and we’re ready to put some wheels on this thing. 

Bots keep running until you tell them to stop (or upon a conditional), so we put a ‘while True’ loop in our main function.

```python
def main():
    previous_df = pd.DataFrame()
    while True:
        # scrape
        current_df = scrape_nfl_gamelines()
        # report differences
        diff = df_differences(previous_df, current_df)
        logger.info(diff)
        previous_df = current_df
        # add to .csv if changes have been made
        if not os.path.exists('dk_lines.csv'):
            save_to_csv(current_df, 'dk_lines.csv')
            logger.info("dk_lines.csv created.")
        if isinstance(diff, pd.DataFrame):
            save_to_csv(current_df, 'dk_lines.csv')
            save_to_csv(diff, 'changes.csv')
            logger.info("Changes saved.")
        # sleep
        logger.info(f"Waiting {wait_time}s...")
        time.sleep(wait_time)
```
Here’s what’s happening… 
* Scrape the game lines from DraftKings 
* Look for differences between the new lines and the previous lines 
* If there are changes, update the .csv’s 
* Sleep for 30s and do it again

Voila!

![Image](https://i.imgur.com/IplaHyh.png)

If you want to do about 100x more work and have the same tools as the pros... 
* Add other sites 
* Add player/team/game props 
* Add in-game bets 
* Put it all in a relational DB 
* Deploy as containers on a virtual machine 
* Optional: Put it in a web app or desktop app 
* Optional: Alerts on number thresholds 
 
 \
 \
 \

## NBA DFS - Part 2 - 9/23/20

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
7,513th: $5.71 

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

L1: 0.56 \
L1: 0.56 \
L2: 1.11 \
L3: 1.11 \
L4: 1.11 \
L5: 1.11 \
L6: 1.11 \
L7: 1.11 \
L8: 1.11 \
L9: 1.11 \
------------ \
Total: 10.0

Notice that even in this dramatic example nobody is winning if rake is 15.6%. I believe the best you can do is 9-10% rake in the $100-$500 buy-in range, and the NFL $3,000 Luxury Box rake is 5.6%. 

I’ll make more DFS posts in the future -- I still want to cover what types of players are high/low variance and share some coding stuff I’m doing.


*In real life, the duplicated lineups will probably have slightly better projections, hence their popularity, but it won’t make up for the dupe effect.

\
\
\

## NBA DFS - 9/13/20

We all know the NBA is the only professional sports league worth paying attention to. We know that! And we also know we can take a few dollars out of the bank and light it on fire at draftkings.com. It’s easy! Kids, [gambling is all around you!](https://www.youtube.com/watch?v=4eHMgXlugIU) 
	
![Image](https://imgur.com/RDZ5f55.jpg)

A few years ago I tried daily fantasy basketball, hoping to turn it into a side hustle to complement poker. I won a little bit, but I was spending hours per slate fine-tuning projections and looking at correlations and spending so much mental energy on it that I was stressed out all the time. I quit after about half a season of playing seriously. 
	
Fast forward to a couple months ago. The NBA, being the only smart and good league, puts their players in a bubble so they can give the fans the playoff basketball they need during COVID-19. I’m learning Python and focusing on data, and I realize I now have the tools I need to automate much of my beloved hobby. 
	
NBA DFS projections from pay sites are pretty decent these days, and since part of my goal is to spend as little time as possible on DFS, I decide to start by averaging projections from the top 3 sites rather than creating my own. It’s kind of annoying to scrape from sites that require credentials, each with different (sometimes difficult) URL paths and weird table formats. All of the sites have one-click .csv downloads, so I find it easier to just bookmark them and download the projections every morning. 
	
Here’s where the fun starts -- my first script has me input the URL of the template for my DraftKings slate and then transforms the projections I downloaded to fit the template. The second script uses the [pydfs-lineup-optimizer](https://github.com/DimaKudosh/pydfs-lineup-optimizer) package to generate the 100 top scoring lineups for the slate. The beauty of using popular pay site projections is that I can  apply linear regression to the projections and lineup %’s to estimate ownership %’s for each player, because everyone else is using the same projections. My ownership projections aren’t bad -- just about as good as the best ones I find online -- and I already know of a couple more predictors that will improve them. 
	
After this all I have to do is bulk upload my lineups to DraftKings and enter contests, right? Well, I could do that, but I wouldn’t win any money. Our goal is to maximize winnings, and in DFS tournaments entering the 100 top expected scoring lineups won’t get you there. I’ll go into detail in my next post.





