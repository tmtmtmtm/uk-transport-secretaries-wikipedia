Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Secretary of State for Transport](https://www.wikidata.org/wiki/Q3246797)
contains all the data expected already — nothing needs fixed up.

Step 2: Tracking page
=====================

PositionHolderHistory already exists; current version is
https://www.wikidata.org/w/index.php?title=Talk:Q3246797&oldid=1112211904
with 22 dated memberships and 7 undated; and 27 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Secretary of State for Transport](https://en.wikipedia.org/wiki/Secretary_of_State_for_Transport)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

This required quite a bit of tweaking to get working. There are lots of
different tables, as the position has changed multiple times, and one of
those (1941-1953) is particularly awkward as there were essentially two different
ministers simultaneously. I've only scraped the Minister of Transport,
and ignored the Minister of Civil Aviation, but the way the table is
structured makes this seem that (for example) Barnes had four different
sets of dates. Rather than make the scraper extra-complicated, I tweaked
the resulting `wikipedia.csv` by hand to merge those.

Step 5: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

28 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/90732d1fc88ac/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

15 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/3b4ea31a3deb3/

Plus several suggested corrections:

1. Bill Rodgers replaces: Q1700212 → Q301252
1. Norman Fowler replaces: Q333259 → Q332919
1. Nicholas Ridley start: 1983-06-11 → 1983-10-16
1. John MacGregor start: 1992-04-11 → 1992-04-10
1. Douglas Alexander start: 2006-05-06 → 2006-05-05
1. Ruth Kelly start: 2007-06-27 → 2007-06-28
1. Philip Hammond start: 2010-05-11 → 2010-05-12

Some of these seem to have mismatched source data, so I'll wait to see
what the tracking page looks like before applying.

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q3246797&oldid=1233218746

This has quite a few issues, which I'll need to resolve one by one.

The first is the appearance of Gavin Strang, who was actually the Minister of State for Transport instead. That exists as https://www.wikidata.org/wiki/Q15109454, but needed a few other claims added.

Strang already has a P39 for that, so I deleted the spurious Secretary
of State claim.

Next is the date overlap between Tom King and Nicholas Ridley. This
appears to be an incorrect start date on Ridley, so I'll accept the
proposed change on that:

    wd uq 'Q324805$93CC316C-66C1-4677-9144-E2AB4AD90941' P580 1983-06-11 1983-10-16

Next is the doubled entry for Norman Fowler. I think is correct as he
was Minister for Transport and then 'upgraded' to Secretary of State for
Transport. This means we'll have to mark him as having replaced himself,
however!

Bill Rogers currently has a 'replaces' of John Gilbert, but that's
incorrect. Gilbert was a junior minister: Anthony Crosland was the last
person in the equivalent role.

So I'll accept the proposed modification for this one:

    wd uq 'Q333259$A96B22DC-5C9C-4461-8A67-E62EA340D8A4' P1365 Q1700212 Q301252

Then we have four people with no dates:

1. Richard Mottram (Q7327922)
1. John Gilbert, Baron Gilbert (Q1700212)
1. Gus Macdonald (Q1555319)
1. Rachel Lomax (Q7279311)

Gilbert and Macdonald were both actually junior ministers, so I've
changed those claims.

Mottram and Lomax were Permanent Secretary to the Dft (Q86689290), not a
minister, so I've updated those as well.

It also looks like I messed up the 'replaces' field for all the people I
munged by hand in the awkward 1941-1953 period, so I've also had to fix
them up manually.

Final version at https://www.wikidata.org/w/index.php?title=Talk:Q3246797&oldid=1233234626
