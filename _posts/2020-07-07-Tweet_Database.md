---
layout: post
title:  "Tweet Database"
date:   2020-07-07 18:46:30 -0500
categories: jekyll update
---
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Tweets Database
An introduction to available capibilities and data on `hydra.uvm.edu`

### Table of Contents
1. <a href='#section1'>Connecting to the Database</a>
2. <a href='#section2'>Selecting Databases and Collections</a>
3. <a href='#section3'>Queries</a>
4. <a href='#section4'>Available Datasets</a>
5. <a href='#section5'>References</a>



```python
import pandas as pd
from pymongo import MongoClient
import numpy as np
import datetime
```

<a id='section1'></a>

## Connecting to the Database
Connections to the database are established using the `MongoClient`. Before trying to connect remember to forward the correct port to your machine, since you won't be able to access the server directly from off campus. 

The command I use to port forward is `ssh -N -f -L localhost:27016:localhost:27016 home-ti-89`, but you'd have to replace `home-ti-89` with your alias for `hydra.uvm.edu`.


```python
username = 'guest'
pwd = 'roboctopus'
client = MongoClient('mongodb://%s:%s@localhost:27016' % (username, pwd))
print(client)
```

    MongoClient(host=['localhost:27016'], document_class=dict, tz_aware=False, connect=True)


<a id='section2'></a>

# Exploring Available Data
MongoDB organizes data into a heirachry. 
The top-most level is the database, which can contain a number of collections containing documents.
Below is an example database, `tweets` on hydra, which has a number of collections such as `geotweets`, or `one_percent`. Documents in our case are tweet objects.

The image is a screenshot of Mongo Compass [(Download here)](https://www.mongodb.com/try/download/compass), a GUI application to explore data stored in a Mongo database visually. 

![](https://mvarnold.w3.uvm.edu/img/tweets.png)

To programatically see the available databases we can execute the following command:


```python
print(client.list_database_names())
```

    ['admin', 'config', 'fulltext', 'languages', 'local', 'tweets', 'tweets_ambient', 'tweets_segmented', 'tweets_segmented_ten_percent', 'tweets_segmented_test']


The important databases for Twitter researchers will be:
 * `tweets`: General tweet collections, like the `one_percent` sample (1% of the decahose) or `geo_tweets`.
 * `tweets_ambient`: Collections of tweets containing a specific word or n-gram of interest.
 * `tweets_segmented`: A continuation of the `one_percent` sample after 2020, which gets updated daily.
 * `tweets_segmented_ten_percent`: A ten percent sample of the decahose.

To select a database from the client object, we can use an attribute access style, such as `client.tweets`, or dictionary access style, such as `client['tweets']`. Let's see what collections are inside the tweets database on `hydra.uvm.edu`.


```python
database = client['tweets']
database.list_collection_names()
```




    ['trump_tweets_w_rt',
     'obama_tweets_from_handle',
     'trump_tweets_from_handle',
     'geotweets',
     'charlottesville',
     'tweets_from_handle',
     'obama_tweets_from_handle_beta',
     'one_percent',
     'trump_tweet_w_rt',
     'obama_tweets_w_rt',
     'trump_tweets_from_handle_beta',
     'tweets_w_rt']



We can also select collections from this database object using either syntax. 


```python
database['one_percent']
```




    Collection(Database(MongoClient(host=['localhost:27016'], document_class=dict, tz_aware=False, connect=True), 'tweets'), 'one_percent')



In general usage the function `tweet_connect` is all you need to return a collection object and start querying tweets.


```python
def tweet_connect(username, pwd, database='tweets', collection='one_percent'):
    client = MongoClient('mongodb://%s:%s@localhost:27016' % (username, pwd))
    db = client[database]
    tweets = db[collection]
    return tweets
```

<a id='section3'></a>

## Querying
Once a collection of tweets is selected, we can query for documents. For example the `find_one` method will return a document.


```python
collection = tweet_connect('guest','roboctopus')
collection.find_one()
```




    {'_id': ObjectId('5dfac73910db86313a1a4312'),
     'user': {'screen_name': 'ebarrera', 'id': 961201},
     'id': 9264581,
     'text': "I'm having lunch... 'Pain Vigneron', 'Mouse de Fois Gras', Comte Cheese, and Some Ethiopan Dish...",
     'created_at': 'Sun Mar 18 12:18:33 +0000 2007',
     'fastText_lang': 'en',
     'fastText_conf': 0.73,
     'tweet_created_at': datetime.datetime(2007, 3, 18, 12, 18, 33),
     'pure_text': "I'm having lunch... 'Pain Vigneron', 'Mouse de Fois Gras', Comte Cheese, and Some Ethiopan Dish..."}



Normally we want to query some specific subset of the data, so we write queries as dictionaries and pass them to a `find` method to return many documents. Let's find English tweets from Halloween in 2019. 

For an exact match on field, we just pass the dictionary key a value; therefore we'll let `fastText_lang`, the language feild, be `'en'`, e.i. `{'fastText_lang': 'en'}`.

If we want to specify a range, such as to match all tweets from 12:00am on Halloween to the next night, we can pass [mongo operators](https://docs.mongodb.com/manual/reference/operator/query/) in dictionaries as the value to a key. The `tweet_created_at` field is what we want to match, so we send the value for it as `{'$gte':date, '$lt':date+datetime.timedelta(1)}`. Pymongo will except normal python datetime types in queries. Thus our full query is as follows:


```python
date = datetime.datetime(2019,10,31)
query = {'fastText_lang': 'en',
         'tweet_created_at': 
             {'$gte':date, '$lt':date+datetime.timedelta(1)}}
```

Let's look at the `pure_text` field of the matching tweets. We'll run our query, and iterate over the returned pymongo curser. You might want to quit if it runs for too long.


```python
for tweet in collection.find(query):
    try:
        print(tweet['pure_text'])
    except KeyError:
        pass
```

    How conservation dogs help track endangered species https://t.co/fZrFvdFN0h https://t.co/V4oaMubFy7
    Shattered. Season 4. Coming soon. https://t.co/xbJOFWiyH5
    Although a new relationship or creative venture may look appea... More for Leo https://t.co/OFWIAY5MDr
    @known_as_kevin0 @amoreekth maybe staying off of twt wasn’t such a bad idea for kevin 😹
    @izgowon are you what
    @DylanHenshaw ur capitalism, sir
    We love hearing from our patients ❤️ 
    •
    •
    •
    •
    •
    #botox #filler #juvederm #restylane #dysport #xeomin #allergan #fillers #neuroxin #RMedicalSpa #MedicalSpa https://t.co/izRHoVxS07
    no wait it's good this season?
    RT @AnnieRiceStL:When MOGOP consultants and leaders say they're trying to make Missouri go "Full Handmaid's Tale", here we go. State-sanctioned period tracking, brought to you by the state that opposes PDMP and firearm permits and until recently REALID. And kicks kids off insurance. #ShowMeAccess https://t.co/OoyDHbqfO0
    #HousingTwitter... sage advice: ad honimens, tweeted at people you've blocked, and fighting other people's battles lead us nowhere. Let's be better. Chances are, if someone's on housing Twitter they've got their heels dug in- pick battles wisely. #bospoli
    @TAYLORS62916765 @soupreviews1 @Combatw67073811 @DrWolfman42 @SamTrickTreat @abhorrently_urs @thestrickenland @kevin_ktx Is it bad to say I am a little jealous?
    @Wyn1745 @BMcAdory9 Lt Cor &amp; so called whistle blower both worked for Brennon. Both CIA operatives. LT Cor attempted to access/alter "classified" presidential call, it was moved to secure server due 2 leaks. Connect the dots. CIA op playing out. All handled secretly in Schiff Intel committee? Hmm?
    @alcoholwipes @badowlposters Nama, Robata, ramen &amp; salad
    Sharing style with large meat and fish family style dinner plates!!!
    🔥 Hot off the press: “Culture Clash Issue #229 - Ownership between PMs &amp; an awesome news app” https://t.co/ofn4JOyFhP #newsletter #productmanagement #apps #productivity
    We hope it is a spooktacular night tomorrow! Head to #DowntownVernon with your kids for a night of asking local shops THE question!... Trick or treat! Start at Justice Park to get a bag &amp; a map of the #TreatTrail at 3 pm! PC: @DowntownVernon https://t.co/IzmxNYrna4 https://t.co/1lxMeIYnRk
    Be a STAR and see the last showings at Millennium Cinema on November 4, AKA National Candy Day, for Dollar Movie Night! Stop by Heritage Hall on October 31, November 1, and November 4 from 11 AM-2 PM to purchase your ticket. We're BURSTING with excitement to see you all there! https://t.co/Pex3NN1aiL
    RT @LilbittyDeedee: Rt in 5 sec for good luck https://t.co/DlzkUhd3qO
    You don't need any powers to be here with me.
    Just relaxing,and watching @NBCChicagoMed right now #ChicagoMed 🏨🚑
    None rn
    Support this kind soul!!
    @FantasyPros I won’t have to bench anyone because I’m going to miss Michael Thomas, Austin Hooper, Todd Gurley, and Brandin cooks #Screwed
    📦 🎁 💼 
    
     You only get one, which one of these three are you choosing?
    @ArianaGrande piggy smalls never looked so stunning wow https://t.co/yfia4ib9DE
    gotta outthink your environment
    Time will tell Mr Kelly https://t.co/b3cqcQL7yi
    I wish @burgerking would bring back shake em up fries😩🍟🧀
    Why not come check out the hottest show on Earth with me tonight?
    I've new too.
    https://t.co/44SCEvFNke
    RT @DavinciKerry: I’ll shove this down y’all throats till I decide to stop. https://t.co/ZiSh2LwGLp https://t.co/AuDCl6pPwq
    @minyardxva you WILL be sorry!
    @realDonaldTrump @senatemajldr Nope. It started with the whistleblower. You’re all crying like babies
    @PrincessTheaOFC Done💫❤️ate theaaaaaaa!!!!!!
    @best_emptynest @owillis @LacyClayMO1 That's really funny. Really funny.
    
    That description was taken from an ACLU lawsuit in 2015 - i.e. under Obama.
    
    So Obama is a monster, too, right?
    
    https://t.co/9JXsG4kFbE
    📸Last night's #AuthorTalk feat. Dieter K. Buse &amp; Graeme S. Mount! Register today for our next Author Talk feat. Kelly S. Thompson on Nov. 25 from 6:30-8pm in the auditorium. . https://t.co/KoBP1C96RE
    @milkyhongjoong he shrugs. are you?
    This man got my house so Smokey, bout burnt my shit down 😂😂😂
    @chunguskook @quieromor1rme @GrandmaSneaky @taehoeng ugh yes so off 🌈
    Motocross: Two junior Whanganui motocross riders podium finish on season debut in Taupo https://t.co/ILrPme6euH
    Ruth Cockburn: Love Letters from Blackpool on Fri 22nd November 7:30 pm https://t.co/e63R6zLt23 https://t.co/nYpKZyKu02
    RT @S1mba__: RT and look at my header or u next😭 https://t.co/dIEcJ6NOuf
    The scene in Burn Without Reading where George Clooney shots Brad Pitt in the head
    @ZeniOliveira8 @Benny4milly Hello kfb
    Go follow my YouTube channel for a free *EXCLUSIVE MINTY PICKAXE* code!
     
    https://t.co/LGNbzrttbZ
    
     #Fortnite #Minty #MintyPickaxeCode #mintypickaxe #mintycode #mintypickaxecodes #fortniteminty #codes #code #fortnitecode #Mintyaxecode #mintypicaxe
    @HanaSongFan Sprays
    You have no idea how much you mean to me and you have no idea how much I'd love to tell you that.
    imagine being involved in student politics like I COULD NOT it's so pretty and meaningless
    RT @todorokenn:MAYBE IF YOU HAD A F*****G BUSINESS YOU WERE PASSIONATE ABOUT, THEN YOU WOULD KNOW WHAT IT TAKES TO RUN A F*****G BUSINESS. BUT YOU DON’T. https://t.co/udZFhtqeP7
    @kayyteee_ 
    GAME
    
    SEVENNNNNNNNN!!!!!
    
    You can reply with #stop to unsubscribe from these notifications, but this is our last reminder. Since ya know, it's Game 7 and all.
    https://t.co/XQsMOJgAMc
    @veramenteam 
    GAME
    
    SEVENNNNNNNNN!!!!!
    
    You can reply with #stop to unsubscribe from these notifications, but this is our last reminder. Since ya know, it's Game 7 and all.
    https://t.co/XQsMOJgAMc
    الْحَمْدُ لِلَّهِ عَلَى كُلِّ حَالٍ All praise and thanks are only for Allah in all circumstances.
    @Vixenfur @yuichurros I love you all are so positive. My hopes is going back 😭❤️
    @beelgives @FortniteGame Yer
    @HammersMemphis looking forward to talking about some good ⚽️ results coming out of the London Stadium. In the meantime, relish in the spectacle that are the Saints 👉
     https://t.co/nFj9df4rf7 #COYI
    Actual facts ‼️
    Learning the Land: Walking the talk of #Indigenous Land acknowledgements: The Conversation https://t.co/sbRRk3ohxR #environment
    
    MORE w/ EcoSearch - news: https://t.co/bjTIPPzt9u web: https://t.co/Hm0BHOdhvY
    Kung sakaling mawala nako, 
    Here's a short note to everyone i love;
    An all new episode of #ChicagoMed starts NOW on NBC!!! @NBCChicagoMed
    a7a i just watched this episode right now



    -------------------------------------------

    KeyboardInterruptTraceback (most recent call last)

    <ipython-input-34-5d59aa58d224> in <module>
    ----> 1 for tweet in collection.find(query):
          2     try:
          3         print(tweet['pure_text'])
          4     except KeyError:
          5         pass


    ~/anaconda3/lib/python3.7/site-packages/pymongo/cursor.py in next(self)
       1187         if self.__empty:
       1188             raise StopIteration
    -> 1189         if len(self.__data) or self._refresh():
       1190             if self.__manipulate:
       1191                 _db = self.__collection.database


    ~/anaconda3/lib/python3.7/site-packages/pymongo/cursor.py in _refresh(self)
       1124                                         self.__collection.database.client,
       1125                                         self.__max_await_time_ms)
    -> 1126                 self.__send_message(g)
       1127 
       1128         return len(self.__data)


    ~/anaconda3/lib/python3.7/site-packages/pymongo/cursor.py in __send_message(self, operation)
        976             docs = self._unpack_response(response=reply,
        977                                          cursor_id=self.__id,
    --> 978                                          codec_options=self.__codec_options)
        979             if from_command:
        980                 first = docs[0]


    ~/anaconda3/lib/python3.7/site-packages/pymongo/cursor.py in _unpack_response(self, response, cursor_id, codec_options)
       1065 
       1066     def _unpack_response(self, response, cursor_id, codec_options):
    -> 1067         return response.unpack_response(cursor_id, codec_options)
       1068 
       1069     def _read_preference(self):


    ~/anaconda3/lib/python3.7/site-packages/pymongo/message.py in unpack_response(self, cursor_id, codec_options)
       1461             :class:`~bson.codec_options.CodecOptions`
       1462         """
    -> 1463         return bson.decode_all(self.payload_document, codec_options)
       1464 
       1465     def command_response(self):


    ~/anaconda3/lib/python3.7/site-packages/bson/objectid.py in __init__(self, oid)
         81     _type_marker = 7
         82 
    ---> 83     def __init__(self, oid=None):
         84         """Initialize a new ObjectId.
         85 


    KeyboardInterrupt: 


Well, those sure are tweets. Let's find some more that are specifically about Halloween.
To do this, we'll use the text index that's been built on this collection. 


```python
anchor = 'trick'
query = {'$text':{'$search': anchor},
    'fastText_lang': 'en',
         'tweet_created_at': 
             {'$gte':date, '$lt':date+datetime.timedelta(1)}}
for tweet in collection.find(query):
    try:
        print(tweet['pure_text'])
    except KeyError:
        pass
```

    RT @hstyleswomen:update || a petition signed by 6.3 billion people has been going around claiming that they'll not be celebrating halloween if harry styles doesn't tweet trick or treat people with kindness again. other one billion people are devastated by this step of the harry styles nation. https://t.co/xEeZK47Vrx
    Trick.....
    And if you’re trick-or-treating in the neighborhood be sure to check out the Ohio City Trick-or-Treat map!
    
    https://t.co/czsQDrt9mz
    @jentrification The trick or treaters in your family? Or the ones coming to your door? We haven't had any trick or treaters at our door. Yet.
    *on trick or treating*
    
    Bea: “Til what age can you go trick-or-treating? Are you ever too old?”
    
    Kai: “It’s never too late!! I don’t believe there’s an age limit!”
    
    #HeSaysSheSays
    #FollowTheSound
    Its halloween and time for punishment and rewards. TRICK OR TREAT. 
    
    TRICK- No limits, hardcore, choking, rough, dominant, controlling, etc. 
    
    TREAT- Loving, gentle, cuddling, romance, passion, etc. 
    
    Dm either one for your reward or punishment. https://t.co/hGv9mEooyC
    Horoscope for today:  "TRICK: The last thing you thought would happen will be the first thing that does."
    
    Ha! Reverse trick: my anxiety has already anticipated every possible scenario, so the last thing must be something good then. 🙃
    Halloween 🎃 trick or treaters Finn , left, aka The Royal Skeleton Guard and Jackson, aka Cuphead went trick or treating in our neighborhood on beggars night. The twins did well for the cold 35 degree temperatures.… https://t.co/bIgQG9Cy8f
    The Candy Queen is ready to greet all Trick-Or-Treaters on Halloween with sweets, smiles &amp;
    toothbrushes!! 😁
    Trick Or Treat Brush Your Teeth! 🦷 
    😁
    Toothbrushes courtesy of Dr. Mitchell Bayroff on Maple St in downtown… https://t.co/5BlizYm1s7
    @UlalaHunterlife trick
    trick or treat
    Trick or Meat
    Trick or Guilty...
    TRICK OR TWEET
    Trick after Treat
    @whatdababe trick or treat
    @vickyveritas Trick or treat
    @mrjamesob Trick or Treat?
    @SWWKF Trick or Treat🎃🍭
    @thearcanagame trick or treat!
    @billy_cfc7 TRICK OR TWEET
    @yuki0129_ITF Trick or treat!!
    Trick or treat @chuzefitness
    @JoeatDawn Trick or Treat ride?
    @iHeartRadio @tiffanyyoung Trick or Treat👻
    Does anyone ever pick "trick"?
    Trick or treat bihhh lol
    Boo!!
    Trick or treat 👻
    Trick or..... https://t.co/MBqJ82CGrp
    Ho HO HO Merry trick-mas
    @ChronoKatie Rip trick or treating. Happy Halloween
    Trick or... @Harry_Styles https://t.co/Su5gTF2Xl3
    trick and treat https://t.co/W1KazGO4kK
    trick or treat https://t.co/u5Fim5EKDf
    RT @nbcsvu: Surrender your candy. 🍭🍬 https://t.co/AeEkJNyztX
    Trick or Treat／MAN WITH A MISSION
    
    #NowPlaying
    @CompoundBoss It’s a trick or treater
    He’s off to trick or treat!!
    @fohti Where u trick or treating tn
    Same old inhouse trick of Soft Pedaling !
    Trick or treat? https://t.co/m7NEOgcnxH
    NL where yall trick or treating at?
    @jeunip Trick #BANGDEKTOT https://t.co/PoILMiXzON
    Trick or treating was pretty lit back then
    Trick or treat? 🎃 https://t.co/pXrFXBvukN
    Who trick or treats at five o clock?
    @JanitorSuper Treat me or I'll trick you
    の略だよね
    RT @danilic: Trick or treat https://t.co/bKwPFVbKsA
    Wish I could take my dog trick or treating
    Trick or heat 🔥 #halloween2019 https://t.co/TzY6FZymfI
    @LewisCapaldi You’re trick or treating tonight in my area??
    lmao my grandma giving out tamales for trick or treat
    RT @mullane_ryan: If getting fucked by Austin Wilde would kill me, I would still do it HAPPILY https://t.co/czEly3X2k1
    Ain't nothin' a trickster loves more than a trick!
    @ladygaga girl this better be a trick on this halloween
    Pretty excited to take my honey trick or treating today ☺️
    NEW ON OUR BLOG: Trick or Treat https://t.co/miaOuI8Enh
    Finna go knock on the plug door like “trick or trees”
    Peak time seems to be over here, 77 trick or treaters
    Lauren really almost just ran over a trick or treater... 😧
    too old to trick or treat &amp; too young to die 😔
    Can’t stop thinking about it. Planning Trick or Treating around it.
    Yalll wana be nasty infested porn stars bout to really trick 😂😂😂😂
    RT @gslayen: Trick or treat as they say 👀🎃👻 https://t.co/YEtx5t4xJF
    Trick? Or #treat? Doesn't matter, party on! #HappyHalloween https://t.co/irGZKNSZ9T
    Trick? Or #treat? Doesn't matter, party on! #HappyHalloween https://t.co/XjreI8Em3J
    My first trick-or-treater was by herself. That hurts my heart. 😢
    Which naughty mom wants to come trick or treating at my house? 😈😈
    hiiii i’m trick or treating can out see in this mask whatsoever
    @TheSommerset kayla wanna go trick or treating with me and then trade candy :)
    @ToySuperMcNasty 😂 lowkey highkey I thought about doing a lil trick or treating
    HAPPY HALLOWEEN 🎃🎃🎃
    Her 1st TRICK or TREAT 🎃🍬🍭 https://t.co/Mw6ETRdNUU
    Y’all I found a kid to take trick or treating tomorrow 😂hahahskdl
    my grandma really asked me “is tmrw trick-or-treat?” ma’am halloween???
    @coinspotau Trick or tweet! (See what I did there..) https://t.co/7n5PJTdlie
    RT @j_mcelroy:Montreal changing Halloween
    
    B.C. changing Daylight Savings Time
    
    If time is suddenly up for negotiation than I have some demands for our elected officials. 
    
    1) Ban Wednesdays
    2) Brunch goes until 4pm
    3) St. Patrick's Day on Saturdays
    4) The month of November? VERY negotiable https://t.co/XfCd3XvuLO
    I'm going trick or treating tonight,call me childish if you want x
    #HappyHalloween2019  a terrorific day many candy minus trick
    🎃🍂🎃🍂🥰😍🧙🧙🏽‍♀️ https://t.co/mJMrMgPBkt
    @partypoops i.... did not know trick or treating was a thing in sg lmao
    sneaks toward @vo_citus and suddenly picked him up.
    
     "Trick or treat, Papa!" she beamed.
    Trick or Treat! Dress &amp; necklace @AngelBrinks @AngelBrinks_Biz #happyhalloween #spookyszn #zombiebride https://t.co/QZXKvccSxs
    Considering dressing up as the BRT and trick-or-treating in Northridge. Scary enough maybe?
    @River_Vox I scolded the news last night "They're moving Trick-or-treating, NOT Halloween"
    @gonzalezgabbiee are you guys gonna go trick or treating or is she still too young?
    everybody bringing they kids to the hill on the trolley to trick or treat 😬🤣
    And settled!
    
    Cleveland let down 3 whole barrels of candy for the trick-o-treaters!
    "and fifa said wanting to laugh was like a trick question.. https://t.co/ei10rtoWwq
    My kids reminding me of the sheer joy of trick and treating https://t.co/tyal8Gn2Ci
    Anyone trick-or-treating Downing Street dressed as a ditch tonight? #DitchJohnson #DitchBrexit #DeadInADitchDay #GE2019 #unbrexitday
    Happy Halloween Everyone, Oishin and his friends heading off trick and treating https://t.co/SJ0FatuM3r
    if you see me trick or treating tmmrw, mind your business!! i just want candy.😭
    Vote for @OwnedFotze in the MV Trick or Treat Contest @manyvids!!
    🍭🍬🎃🍭🍬🎃🍭🍬🍭🍬🎃🍭🍬🎃 https://t.co/tThUhql8JO
    @CashApp This has been a trick full day and this treat would be a answered prayer ❤️
    Baby toes to the face while nursing... trick or treat? 🙃 #breastfeeding #gymnurstics https://t.co/gEnqh1mlpX
    RT @sahouraxo: Inspiring the children of Iraq, Afghanistan, Libya, Syria and Yemen... by bombing them. https://t.co/L0p5cLE8q7
    Does everyone’s neighborhood start trick or treating early af like 6pm??? Or is that just mine??
    RT @sahouraxo: Inspiring the children of Iraq, Afghanistan, Libya, Syria and Yemen... by bombing them. https://t.co/L0p5cLE8q7
    RT @pattonoswalt: Donald, those are just trick or treaters. Put the net down. https://t.co/GwEhEKBXgm
    My kids are not going trick or treating my son sick it’s to cold for them
    @V_actually @KattLivesMatter So nice for the children to see this while trick or treating!!! What a dick!
    #newmusic 'Trick Or Treat' - Ep Mixtape by o'g wizard #halloween https://t.co/tOHLx69cZF via @DatPiff
    @skinhub halloween🎃 is coming... any trick or treat for me 👻 
    happy halloween https://t.co/48XXqWTfGD
    RT @HongPong:It is pathetic that the usual drug scare clickbait stories have been rolled out for audiences in the Fear Zones.
    
    Vehicle collisions, not bad candy, are the real serious threat on Halloween https://t.co/ywu7ixXfWO
    I do know it’s gon be to damn cold to take my son trick or treating 😂😂😂
    Halloween Weather: Trick-or-treaters advised to get out early https://t.co/kJg8xf4gcs https://t.co/VadS42OzpR
    Some pics from our trick or treat field trip to Presbyterian Village in Homer! https://t.co/FplCqh7gcO
    ⚠️🚨 IMPORTANT ANNOUNCEMENT 🚨⚠️
    Look at my dog waiting for trick or treaters https://t.co/0n73i0B7Wh
    Ready for lil’ trick or treaters, courtesy of Clara and Elliot’s pumpkin carving. https://t.co/4W3XjHigSK
    @FLOTUS @WhiteHouse The trick was the HOUSE passing impeachment proceedings
    
    The treat will be when @POTUS orders #DECLAS
    Good early Halloween trick or treat time with the boys! #Halloween #SoManyBoys #TrickOrTreat 👻🎃 https://t.co/XgemXp2BMX
    Trick or treat? Definitely a sweet treat from spice it up exploratory! #prmsrocks @RyonMiddle https://t.co/r5shhMwlLh
    Trick or Treat your hair was a success! Thanks for joining Impressions Of Beauty #MyASU https://t.co/CqEygqa4Dw
    Something spooky has happened at Station View, is it a Trick or a Treat🧟‍♀️🎃👻 https://t.co/ecwzU0y46z
    I am so excited to have trick-or-treaters for the first time now that we have our house
    Sex is great and all but have you ever gotten a full sized candy bar while trick or treating?
    Happy Halloween on behalf of the #dynamicottawa team
    
    Make sure to stretch before trick-or-treating! https://t.co/f88vbVuh0P
    @JIHYOPOP i saw a group of furries when i went trick or treating last year ✋🏼🤡 https://t.co/mK5o3MIjXE
    @WriteinBK "Let"? Folks identifying trick or treaters? They profiling? If you've got candy &amp; kids come, give them candy.
    When Brantley Finch goes trick-or-treating this Halloween, he will be sporting an epic costume. https://t.co/zpKsWzMTCw
    Trick or treating Android Emoji keyboard app makes millions of unauthorized purchases https://t.co/Fy4c2x6Clt https://t.co/PAoBhQUTf8
    Trick or treat? Happy Halloween for an hour and a half or so 🧡 #halloween2019 #trickortreat https://t.co/1NjiA2ycXM
    If you're in Brooklyn and reasonably close to Park Slope, take your kids down there for trick or treating.
    My kind of trick or treating. Kids get candy, mommy gets a beer from my neighbor. https://t.co/G6fFFWMATb
    I hope you can slip away early, Hop - we can team task answering the door to the trick and treaters.
    Vote for @MissNovaStarNL in the MV Contest Trick or Treat Contest @manyvids https://t.co/YHFMvZk5DG https://t.co/SVLcQUCRPa
    Zatanna says she was promised trick or treaters. She doesn’t understand that Manchester postponed Halloween. https://t.co/3pPDDfjqMo
    Trying to get home in time for the Trick-or-Treaters? Order III Forks for delivery! https://t.co/77Vn6Zl09r
    Distracted and drunk drivers pose dangers for trick-or-treaters - Oct 31 @ 1:18 AM ET https://t.co/zXKcTkb11g
    And if a teenager is choosing trick or treating over partying, causing mischief, etc. they deserve a freaking candy bar!
    Halloween parties are over! Now to hang out at home until trick or treat time! 🎃👻🎃👻 https://t.co/BCegJewgws
    We have severe thunderstorms and a tornado watch for Halloween night. This feels very unfair to the trick-or-treaters tonight.
    Before you trick or treat, please submit your time sheet! Happy Halloween from the Payroll Office 🎃👻 https://t.co/kRJi8x6rM7
    RT @JohnHal29271650: I was told she has a mattress strapped to her back incase she meets someone she knows.🤥🤥😂😂 https://t.co/rFdhGP0mS6
    RT @BlackYellow: *Julian Brandt has entered the chat* https://t.co/GInMn2i1Bp
    RT @APEntertainment: TRICK OR TREAT: @AndersonPaak says give him anything but candy corn and Milky Ways.🍫 https://t.co/ikDlDLS6CJ
    Lowkey wish i was working so i could dress up and see the cute kids trick or treat around the mall 🥺
    Google turned its homepage into a spooky trick or treat game to celebrate Halloween
     https://t.co/8TSZ1BqKjy https://t.co/g9St8AUOaC
    Adults that actually enjoy and look forward to trick or treaters coming to their front door all night are borderline paedophiles. Discuss
    Happy Halloween! Hope you all have fun being spooky, trick and treating, or however you spend this awesome holiday! 🎃 👻 🍬
    @tobadzistsini Yes. It's very annoying. I bet it cuts our trick-or-treaters by 50%! (From about 6 to about 3.)
    @xSnuwu True but i dont want to have to shovel! And i feel bad for the kids trying to trick or treat!
    i dont really like candy + i can’t trick or treat bc i’m an actual adult so it doesn’t matter 😎
    Trick or Treason by Kathi Daley
    🎃https://t.co/ba0ngngXTD
    #amreading #booklove #ilovecozies #cozymystery #cozymysteries
    #cozies #cozymysterylovers #cozymysterybook #CelebratingCozy https://t.co/5pp52OjEWa
    @OhThatsEm Ask me why my parents just asked me if I'm going trick or treating like whaaa...must be a Polish thing
    I have already gained 10 lbs at work eating candy and I havent even taken the kid trick or treating yet. #halloween2019 #trickortreat
    Trick or Treat! 👻 Our favorite holiday is here residents and we want to know how you are celebrating? https://t.co/NQM6wTXMDf
    Halloween trick-or-treaters facing ‘big problems’ as storm targets East with severe weather threat https://t.co/N5LM0o5h3u https://t.co/IMmdURKAdn
    You may be too old to trick or treat, but you can always treat yourself to something sweet with us. https://t.co/rn9xKtdc3X
    I won’t be home to greet the trick-or-treaters but lmaooo y’all think this will do 🥴 https://t.co/8sVHUEp4CO
    My generation used pillow cases for trick or treating.. by end of the night the whole pillow case would be filled with candy 😂😂
    RT @Goodtweet_man: Yes you shitty asshole https://t.co/N9n66UoDgL
    RT @bourgeoisalien: if this is your biggest complaint in life, you absolute bourgeois ghoul: fuck you
    
    hope that clears things up https://t…
    RT @bourgeoisalien: if this is your biggest complaint in life, you absolute bourgeois ghoul: fuck you
    
    hope that clears things up https://t…
    Happy Halloween!! We hope everyone has a safe and fun night of trick-or-treating 😁 🎃 😁 🎃 😁 🎃 https://t.co/0zgtPr0Ffb
    HAppy Swallow-ween 👅 make sure you get that treat 🍆 and turn around and trick that dude and steal his credit card 💳 🎃
    RT @BethSkw:YES omg I'm sorry there are far bigger things to be mad about on Twitter today but if you're going to give candy away, you are in the game of being generous. You are not the candy police. The world does not need candy police. https://t.co/96acrwNYFK
    RT @thekjohnston: Yes. Yes, you do, you selfish elitist asshole. https://t.co/PD3gL6VXDq
    The one thing I love about my neighborhood?  No trick-or-treating kids.  Which means no doorbells fucking with my dogs and birds all night.
    Trick or treating could be a little chillier this year. @JackmanWx has a look at Halloween weather and your weekend forecast. https://t.co/ovQHb5ZTLz
    im about to go trick or treating!! im rlly excited but also rlly nervous bc i dont want adults being assholes bc im 18 asdafafs,,,
    Happy Halloween everyone. 
    Enjoy trick or treating and the fireworks . STAY SAFE and HAVE REGARD FOR ELDERLY NEIGHBOURS AND their PETS. https://t.co/BK7pFa7oIZ
    RT @Goodtweet_man: Yes you shitty asshole https://t.co/N9n66UoDgL
    Happy Hallowen 🎃 We had a fab time on our Halloween cruises on Saturday!
    Stay safe out trick &amp; treating tonight 🍬🍫🍭 https://t.co/ja3JRzf6xD
    RT @AndySN94: When the trick or treaters finally fuck off after knocking 10 times and you’re hiding behind the couch in the dark https://t.…
    @zurqab @supercuddly @mishtal @vadid65 Bit of a one trick pony aren't you zurqab ? You're the moslem, so explain if you can. Allah is watching.
    Buddy! Surprised!! ㅋㅋㅋ YujuNa is ready for halloween ㅋㅋ you guys surprised, right? We Succeed in surprising you guys! ㅋㅋㅋ Trick or Treat! https://t.co/bqUXUVmPOo
    Halloween party:
    
    @Crazy_Mama_G @FireGoddessB @MrsRabbitResist @zelda229 @sdr_medco @PatriotsGirlUSA @RhymesRadical @Geno1955 @vegix @Gr3Te4rights @Rose52413 @DearAuntCrabby @fairfield12 @michelle_spenc @JustKathyRay @RENEEWEATHERS2 @RosannaPhillip 
    
    TRICK or RETWEET 🎃 https://t.co/XhLp8JVy56
    @WFKARS I can’t...believe this is real? We live in a GIANT neighborhood and are prepared every year...have never once had a trick or treater. ☹️
    Another great year at Shawnee Safe Trick or Treat! Photos available to view and purchase on Shawnee Photo Club website: https://t.co/QqqorieBmr https://t.co/hFzFaNYnc1
    I’m so high I’m thinking about how worried I am that I’m going to say “merry Christmas” when the trick or treaters come to the door
    @papalodown Oh! so Montreal has a fierce rainstorm coming today so the mayor told everyone to save their trick or treating for Friday, so it's only a local thing
    Check out my Gig on Fiverr: professionally proofread and edit your document https://t.co/lkbapubgff @Wonho @Happy Halloween @Monsta X @taciz @Han SeoHee @BritainHasExploded @Monbebe @starship @trick or treat
    When mom helps you with your Princess hair and dress, and you love the look.... A pirate and a princess looking forward to Trick or Treating tonight! https://t.co/Rn7NEiTaj1
    @nhinxie Give them to your poorly behaved God Son who wasn’t allowed to go trick or treating, due to the fact he refused to wear clothes because he was hot.
    So excited for my halloween tradition of buying a bag of candy, getting zero trick-or-treaters, then eating the bag of candy alone. This time it's laffy taffy.
    
    #TrickOrTreat
    @AshWilke Have a safe, fun , Halloween, Ash! 😊tomorrow I’ll send you or post a picture of myself as i greet the kids coming to our door for trick or treat!
    My children are adorable. An enderperson from Minecraft, Rainbow Brite, and my dad, who plays the bagpipes as the children walk from house to house trick or treating :) https://t.co/piQb0wgRWS
    Boomers : ugh you're too old to trick r treat!
    
    Me, giving anyone that comes to the door a handful of candy:
    
    O K A Y   B O O M E R
    Halloween 2019 looks like this...
    🎃 
    Jasmine and friends
    🎃 
    Karmadée’s 2nd time trick or treating
    🎃
    Mason - too cool to participate this year. 😎 
    
    #halloweeninaustralia #halloween2019 https://t.co/Zqm78g3iWG
    In ten years at this address we have had one single trick or treater.
    
    It was, naturally, the year that we had no sweets or confectionery of any kind in the house.
    BOO! It's Halloween - please be on the lookout for children who may be trick-or-treating this evening. Have fun and stay safe and dry this evening! https://t.co/j0FbmWIBSJ
    @emma74748 SO I have two demons outfit! And a wolf outfit! (I am going out trick or treating as a wolf!) Lastly a parrot outfit! 
    Happy Halloween!🤡 https://t.co/Nwkmx3HvMf
    @neontaster Stairway to Heaven - Led Zeppelin / Dolly Parton
    
    But also:
    
    Wichita Lineman - Glenn Campbell / Dwight Yoakam / REM
    
    I Want You to Want Me - Cheap Trick / Dwight Yoakam
    
    Suspicious Minds - Elvis / Dwight Yoakam
    "Ehem... Succubi aren't SLUTS or WHORES... We act that way DELIBERATELY to trick mortals into giving us lewd energy...
    
    Behind closed doors, we're VERY sophisticated... I particularly like to play Chess"
    Happy Halloween, everyone!
    
    Do not get hit by cars when trick-or-treating, that might be bad for your health
    
    Lol, I'm hilarious-
    
    But seriously, don't get hit by a car-
    Pumpkin carving and ghost hunting. Trick or treating and witch hunting. This will be a bone-chilling Halloween! 
    Happy Halloween to all!
    #holisticveterinaryhealing #veterinary #animals #dogs #cats #animallover #doglover #catlover https://t.co/C4mhJAuhNe
    Fun Halloween trick: Go to the supermarket and buy a big bag of apples and two packs of razor blades, then ask the checkout girl out on a date. Just to hear the scream.
    RT @BlueCollarBret:Genius.  If you are too old to trick or treat but still want the experience and/or the candy, no problem.  You just have to be creative. 🤣 https://t.co/pBzR5czLzK
    The sun has gone down.👻 Time to go "Trick &amp; Tricking"🎃👅🏳️‍🌈 #HomosexualChristmas
    @CowboyInt 
    @Iz_xfun
    @PozitiveVizion
    @TuckyLucker 
    @jahdoughboy
    @WesPipes4
    @geo257764
    @fungokid
    @AussieManGay
    @BmPhil75
    @RedBear35
    @HarrySmee
    @hotcountrydudes 
    @Otis_1988 
    @dann00dann 
    @lastoeck https://t.co/xQTc73sNnK
    Temperatures are finally expected to begin cooling down in the Pensacola area this afternoon, just in time for trick-or-treaters to take to the streets and beg for candy tonight. https://t.co/y2bf9x850B
    IF we get trick-or-treaters tonight, I have a feeling they will not be amused with Dana’s tiny cheez it bags. There aren’t even 10 cheez itz in each 😂🙄 #sorrykiddos 😢
    Wishing you all a HAPPY HALLOWEEN!! 🎃👻 
    
    We are advising all trick or treaters to only knock at doors of houses which are decorated - this way everyone can enjoy Halloween! 🎉 https://t.co/GS40NiJqLy
    Possibly the quietest trick or treat night I've had, everyone must be away for half term. Also very, very polite children only taking one sweet, for the first time ever I may have some left!
    I've been smoking weed since I was about 14 years old you'd think I'd know a trick to stop the smoke from getting in my fucking eye by now this shit hurted 😤😭
    Never forget that young Michael Myers went on his inaugural killing spree right after getting a bunch of candy corn in his trick or treat bucket.... I’m just out here trying to save a life
    My Halloween celebration is still going on, everyone! Make sure to DM me "Trick-or-treat!" To get a nice bundle of hentai! 💖 ~ 💕 #Hentai #Lewd #Ecchi #halloween2019 #trickortreat
    
    @Kirazelb45 @Feetloveralways @DemonMonika666 @LokinaSama @Hentusiast @Dv8Hentai @DoujinsApp
    @CardsPA Unless you moved I’m coming back to your house!  I’ll be the guy dressed up as an old guy to old to trick or treat!  Maybe to old to make it around the circle!
    Historically, candy meant for young consumers has sported poisonous-sounding, WTF wrappers and packages that most self-respecting parents would be dismayed to see dumped out of their children’s trick-or-treat bags. https://t.co/kLrzXZ7qqv
    Thank you to the adorable trick or treaters that stopped by our office. Instead of taking candy, they handed out delicious treats to the entire staff.
    
    Have a Fun &amp; Safe Halloween!
    
    #halloween  #adultslovecandytoo… https://t.co/ZSuwex6kGO
    Political power to commit crime, manipulate Justice system to protect themselves at everyone else's expense.  The oldest trick in the book.  Now, too much at stake.  Let's regroup, get real and be good to one another.
    I don't do many parties, my neighborhood is too old for trick or treaters, between @DraggetShow with their amazing DST3K and this whole story already, my Halloween is made and I feel like it's been duly celebrated.
    When it comes to kids’ health, flavored e-cigarettes are all trick and no treat. More than ever, @POTUS and @FDATobacco must implement their proposal to take these addictive, kid-friendly products off the market. #AllTrickNoTreat https://t.co/BXeDVf0OlR
    our house only gets like three trick-or-treaters a year somehow but now that it's snowy we'll be lucky to see one lone kid
    
    which is a-ok bc I don't think we even bought candy
    it stopped raining and my tia calls me saying she wants me to take my baby cousins trick or treating which makes me HAPPY but also i just took off my makeup and changed into pjs https://t.co/gE3j7znT4O
    I remember as a kid, we knew where to go to get full size candy bars. And Kroger in my town always gave us cans of pop. Kids today have no idea how luxurious trick or treat used to be 😂
    All through the neighbour hood there was not a peep 
    Kids came n went as they trick or treat 
    Every where you looked they were all over the street 
    Meanwhile my girl is in the picture window sucking at my meat
    My sister wants to complain about the time we should go trick or treating and this fckn girl is like "I'm waiting for my bf". Like he ain't your kid sis, tf get in your car and take your daughter
    Children face a greater risk of automobile related injury on Halloween than on any other day of the year. Avoid automobile trips during trick-or-treat hours, and if you must drive, remember to take it slow, avoid distractions, and be vigilant.
    HAPPY HALLOWEEN! The rain won’t stop an astronaut, a party cow and so called Marvel comic superheroine Polaris 😆
    
    Ready for trick or treat...what’s your favourite treat?
    
    I would have to say Oh, Henry - peanuts,… https://t.co/47YRTYO3Oz
    When I was younger my parents stole all the good candy from my night of trick or treating. I got stuck with candy corn and mars bars shit was trash I had to hide my M&amp;M skittles and such things
    RT @Pudkip: EVERYBOYD SHUT THE FUCK UP AND LOOK AT THIS RIGHT NKW OH MY@GOS???? OH YM GOD?? https://t.co/P30w6uYEN7
    Happy Halloween in a couple hours I’ll be taking my little baby brother trick or treating since tonight no power in the town dad lives in I’ll post a photo before we go till then -hugs everyone- https://t.co/R3GlSLzj3R
    @ij_ford I haven’t had trick or treaters call for years but there have been two lots tonight and I had no sweets in! I could only offer them apples which they took- to my surprise! That was nice of them I thought
    @NateMJensen It’s also more popular for the weekend before Halloween. Everyone’s reporting that they’ll trick or treat (weather permitting), but trunk or treat events all happened last weekend. One school had a drug free focus/health fair attached to theirs.
    RT @ClassySaifian:Earlier for Britishers  👉 poor indians home used to be hottest tourist destination for enjoyment then 👉 Indian politicians for seeking votes...&amp; Now a Canadian is using the same trick for his PR... SHAME SHAME SHAME @akshaykumar https://t.co/YXYTwhpsLH
    jungkook bts tik tok 18+ gain mutuals follow trick sugar daddy mommy giveaway gc fancam army fic au angst sub dom seungyoun blackpink kinky furry rt nsfw bdsm dom sub shawn camila lesbian hot smut bts like for like pussy  https://t.co/RRUnnxVGSF
    Happy Halloween! Don't let your students get spooked by #STEMed. Make science and math fun and exciting with ExploreLearning products. Trick or treat yo' self to making your lesson plans easier! #ELGizmos #ReflexMath #Science4Us #elpromo https://t.co/zzsA3FUzij https://t.co/ca3WeY3Say
    I almost forgot to post!  #Halloween #workout!  Had to do short ones today because I was so busy prepping for Trick Or Treating with the kids and Japanese friends who were coming on base to go with us. #MM100 for the… https://t.co/HraHBAyPv8
    Can parents outsource the task of taking their kids trick-or-treating?🎃 Probably not, but they can certainly outsource party planning or driving to soccer practice.
    
    WSB's Amber Epp discusses the tensions parents face in making these outsourcing decisions.https://t.co/7RfNUbsfgy
    @stantal13441147 ITZY - Cherry
    TWICE - TRICK IT
    TWICE - LOVE FOOLISH
    TWICE - 21:29
    NCT DREAM - Dream Run
    SuperM - Super Car
    Stray Kids - Boxer
    Wheein / MAMAMOO - 25
    MAMAMOO - My Star
    Blue - Taeyeon
    Red Velvet - Sunny Side Up!
    Red Velvet - Sayounara (if Japanese counts too)
    GFriend - Starry Sky
    RT @alexxismariee_:Heads up that my brother is a 25 year old non-vocal man with Autism who may be showing up at your door to trick-or-treat. Please be patient and kind if he appears at your door.💙 https://t.co/7mFhVXG6Gq
    “ playing it safe again I see... Fine I'll crush that defense.. draw ”
    
    He grinned as he slapped another card down to his duel disk
    
    “ here's another cyber dragon Drei... And it's effect activates as well.. haha... Now here's my new trick.. –
    Yearly reminder that knocking on random doors can be dangerous. If your kids are getting to the age where they want to trick or treat without the parents, I recommend knowing where they will be and what your local sex offender registry looks like. 
    
    Just sayin.
    Trick or Treat! It's a treat that this book arrived for me today! I adore rhettandlink and their YouTube show. They were one of the first channels I watched on YouTube and still watch them nearly every day so I had… https://t.co/HF8LEYBOCx
    @ShaggyKC @mjbj731 @SharylAttkisson Not to my knowledge, but the neutral gender is the origin of the idea that there can be genders other than M/F.
    
    The sleight of hand trick to replace "sex" with "gender" opened the door to the whole nonbinary universe of insanity.
    ALERT🎃 👻:  it’s too dangerous to be sending your children out to  Trick or Treat 👻 It’s too dangerous to even open your doors for  these masked adults.  NO ITS NO JOKE ! People use the masks to do home invasion... https://t.co/of7VvUOktG
    October 31, 1995:  
    *Raining*
    
    Us kids trick or treated in the rain
    
    October 31, 2019:
    *Raining*
    
    “Oh my gosh it’s going to rain.. The kids can’t go out... Should we move Halloween to Saturday?” ...Discussions all day on October 30.
    
    What have we become?
    @MrTomPaye We just had our first bunch of little trick-or-treaters. Was glad to have a bag of Flake minis for them, but they were more interested in my "6 million cats" (who have the naughty habit of trying to run out when we answer the door). 😂
    Today we are open our normal hours. The only change is that Open Jump will start at 1pm directly after Rockin' Tots.
    
    We are offering a free meal to all first responders as well.
    
    Tonight ware hosting a trick or treat event from 5-8pm. https://t.co/sV86Lzs0Q9
    RT @redsteeze: He’ll be easier to see by oncoming traffic this year at least. https://t.co/69o8HgsobG
    As we are getting ready for trick-or-treating, our friends in the UK are getting ready to celebrate Bonfire Night! Every Nov. 5, they celebrate the safety of the king/queen with bonfires and fireworks displays all over the country!  #Europefacts
    
    Photo cred: BBC https://t.co/OaLlWBPCnN
    RT @Paula55855:How worried are you about this behaviour?  The people wanted to Leave the EU, those in charge didn’t &amp; here we are 3.5 years later, are we about to see a huge magic trick that will result in us staying in the EU? https://t.co/ci1HNHKP6K
    Happy Halloween! What's your favorite candy to find in your treat bag?
    🧡🎃📙🎃🧡
    Looks like a stormy evening for us, which means fewer trick-or-treaters. BUT it also means any kids who show up tonight are getting massive handfuls of candy.
    🍬🍫🍬🍫🍬
    https://t.co/NYcLcZC0QI https://t.co/wxF7zGKl4l
    it was such a roller coaster week, stressful and exhausting. As a treat to myself (not a trick) I'm welcoming long holidays with something I've waited for a long time (and before leaving PH) it's best to celebrate Sugarfree music tonight see you @ebedancel! 😊 https://t.co/eVBVbWQROI
    Almost 730 pm on Halloween, not even one trick or treater. It ends at 8pm here. This makes two years. I go all out and decorate, buy hundreds of pieces of candy. All for what? Mad and very sad.. I know I live in a small town, but not even one kid?
    RT @NWSStateCollege:#RT @NWSStateCollege: The greatest risk for severe thunderstorms will be Thursday afternoon and evening as a cold front moves from west to east across PA. If you're heading out trick or treating, check the weather before you go and have a way to get weat… https://t.co/5sFbKCnlvb
    I was about 9 or 10 when my mom (who was white) let us trick or treat by ourselves for the first time.  It was also the first time i saw houses turn off the lights when we approached.  They hadn't turned off the light the year before. https://t.co/EOh8iEIKAj
    We hope it is a spooktacular night tomorrow! Head to #DowntownVernon with your kids for a night of asking local shops THE question!... Trick or treat! Start at Justice Park to get a bag &amp; a map of the #TreatTrail at 3 pm! PC: @DowntownVernon https://t.co/IzmxNYrna4 https://t.co/1lxMeIYnRk
    I bought too much candy for my students, thinking like my old schools teachers they would just send their kids to me randomly for Halloween. 😂
    I gave every kid who came in several pieces when they said trick or treat, gave some to coworkers to and I've barely made a dent. 🍫🍭🍬
    As it’s Halloween🎃, the next trick we wanted to treat you to is the art &amp; benefits of a positive mental attitude when faced with your ‘to do’ list. 
    
    So here’s @RussellTreasure with a couple of handy hints for you. Hope it helps pumpkin 🎃
    
    #mindfulness #libertaswm #lists https://t.co/P3D9xJwlWX
    Mitchell just told me they weren’t allowed to trick or treat growing up!! 😭😭😭 What a sad childhood! I remember eating dinner as fast I can so we could hurry up and get as much candy as possible! I loved going door to door and meeting neighbors and just getting so much CANDY!!
    I remember me and a bunch of other kids on the street went trick or treating in the middle of summer once and everyone was like wtf but there was an Indian family in the neighborhood who was like "oh this must be some custom we don't know about" and gave us some shit
    Happy Halloween! 💀 👽 👻🦇🎃
    
    I hope everyone will be enjoying this spooky time of the year by either trick or treating with your kids, watching a movie, reading a good book, or just hanging out with your family. I know I am going to be reading and spending the day with my dogs. https://t.co/J78nhhG5MW
    Thanks to those all sent me a reply today, I will get round to you. Just been a manic day. I also did my first trick or treating since I was a kid. Well my son wanted to do it and didn't want to let him down.
    
    Had to be back by the time #TOTP started though. Joke! 🤣
    @DieterFrikadell @Matt_Clough @TheDisproof @fishyfish67 @rumpledrumskin @Over400ppm @RedDragonFly19 @donsmithshow2 @Jamz129 @AugeForTrump @aSinister @OscarsWild1 @BubbasRanch @BardLackey @Stahlzart @JamesRider3 @CymaticWave @PaprikaLady @Schtickery @acuna_r @DawnTJ90 @insane_voice @ranger_ivan @triton6346 @TQMKA @TheDalaiLamaCon @91996340e81d45a @NIMN2019 @annettehunter77 @MKahn84 @BadgersNo @baltree @bfraser747 @TurtleFL @MartinJBern @Stephen90045069 @Xshnargloth_II @Paulward44 @JonLeSage4 @ronin47 @PrimleyJack @CromwellStuff @MATTP1949 @Donelsonfiles "If the graph was about tree rings"
    
    It's literally a GRAPH OF TREE RING MXDs—right up until the late 20C. Yet by the mere trick of writing "temperature anomaly" on the left hand side, they've convinced you it's "about" temperatures. 
    
    You're the prime mark[et] for these frauds.


    Traceback (most recent call last):
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/mongo_client.py", line 1498, in _process_periodic_tasks
        cursor_ids, address, topology, session=None)
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/mongo_client.py", line 1428, in _kill_cursors
        server = topology.select_server_by_address(tuple(address))
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/topology.py", line 247, in select_server_by_address
        address)
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/topology.py", line 224, in select_server
        address))
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/topology.py", line 183, in select_servers
        selector, server_timeout, address)
      File "/Users/michael/anaconda3/lib/python3.7/site-packages/pymongo/topology.py", line 199, in _select_servers_loop
        self._error_message(selector))
    pymongo.errors.ServerSelectionTimeoutError: localhost:27016: [Errno 61] Connection refused


<a id='section4'></a>

## Available Databases

### 1. `tweets`

Most simple tweet collections are stored in the `tweets` database.

##### Collections:

 - `one_percent` - A one percent sample of the decahose containing tweets from 2009 to the end of 2019. 
     Sample is continued after 2020/01/01 in the `tweets_segmented` collection so we can keep everything up to date.
 - `geotweets` - A collection of tweets with lat/long coordinates.
 - `charlottesville` - A full decahose sample for the time periods around the Unite the Right rally in Charlottesville, NC.
 
### 2. `tweets_segmented`

A continuation of the one percent sample, seperated into month-long collections. Begins in January, 2020.

### 3. `tweets_segmented_ten_percent`

A ten percent random sample of tweets, in the same month collection format as above.

### 4. `tweets_segmented_location`

A database of all tweets containing user location metadata that matches a US city. Divided into collections of one year. 

The user information is not time accurate as in the `geotweets` collection, but rather is the location given by the user when they created an account. 

### 5. `tweets_ambient`

A database of tweets containing a particular $n$-gram. Currently one collection exists: `mental_health`.

<a id='section5'></a>



# References
[Introduction to MongoDB](https://docs.mongodb.com/manual/introduction/): A good starting place to understand the structure of the database.

[Pymongo tutorials](https://pymongo.readthedocs.io/en/stable/tutorial.html): For learning python syntax for interacting with the database. The main docs are written for the Mongo Shell.
