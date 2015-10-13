Configuration
=============

Start with a list of shows, and their Zap2It Ids (from
`http://tvschedule.zap2it.com/`):

  %shows = (
    'EP00931182' => 'The Big Bang Theory",
    'EP01569372' => 'Arrow',
    'EP01279298' => "Bob's Burgers",
    'EP01739009' => 'Brooklyn Nine-Nine',
    'EP00750178' => 'Doctor Who',
    'EP01566290' => 'Gravity Falls',
    'EP01158124' => 'Modern Family',
    'EP01922936' => 'The Flash',
    # etc
  );

This can be hardcoded into the script for now.

Data Storage
============

Use SQLite as the storage format; inferred schema below.

Data Collection
===============

For each show, append "0001".  Check if it's in the database, otherwise fetch it
from `http://tvschedule.zap2it.com/tvlistings/gridDetailService?pgmId=$ID`,
strip to the first = , and parse it as json.  Increment the counter and try to
fetch the next one.  When you get a response without a seriesId, stop; you've
reached the latest episode.

Returned Data
-------------

    zctv.gd.schedule['EP012792980093'] = {"program": {
        "description": "When Louise reveals that she hasn't been scared before, the Belchers visit a haunted house; the trip is more frightening than anticipated.",
        "genres": [
            "Animated",
            "Sitcom"
        ],
        "seriesId": "EP01279298",
        "originalAirDate": "1445169600000",
        "episodeUrl": "http://tvlistings.zap2it.com/tv/bobs-burgers-hauntening/EP012792980093?aid=zap2it",
        "schedule": 0,
        "episodeTimesUrl": "http://tvlistings.zap2it.com/tv/bobs-burgers/upcoming-episodes/EP01279298?aid=zap2it",
        "type": "sc",
        "flagship": true,
        "seasonNumber": "S06",
        "episodeNumber": "E03",
        "duration": 0,
        "isMyFave": false,
        "canMyFave": true
    }}

Inferred schema
---------------

    CREATE TABLE episode (
        episode id
        title           # comes from schedule page
        description
        season
        episode
        original air date
        episode url
    )

    CREATE TABLE schedule (
        episode id
        datetime
        timezome
        channel
    )

The Schedule
============

For each episode, fetch `http://tvschedule.zap2it.com/tv/*/$ID?aid=tvschedule`
and extract `//li[@class="zc-sc-ep-list-l"]`, which contains the following
things:

    <li class="zc-sc-ep-list-l zc-sc-ep-list-wd">Mon</li>
    <li class="zc-sc-ep-list-l zc-sc-ep-list-md">10/19</li>
    <li class="zc-sc-ep-list-l zc-sc-ep-list-stet">8:00-8:31pm</li>
    <li class="zc-sc-ep-list-l zc-sc-ep-list-call">CBS</li>

Parse and compose into a datetime (and possibly a channel).

Title also comes from this, at `//span[@id="zc-program-title-ep"]/text()`,
but only use this if the item doesn't already have a title.

