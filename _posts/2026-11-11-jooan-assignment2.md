---
title: "Assignment 2: Reading the Route, Not the Name. Manual vs. Automated Geocoding of 1 the Road"
permalink: /assignment2/
last_modified_at: 2026-11-11T23:30:00+04:00
tags:
  - Recogito
  - Geocoding
  - NER
  - Interactive Map
  - R
---

## Introduction

Before I opened the map, I thought I knew how the ranking would turn out. The manual annotations that my classmate Nahla and I made in Recogito would be the most accurate, and of the two automated methods, the LLM would beat spaCy, because spaCy reads words exactly as they are written while an LLM can, at least in principle, read context.

The map did not agree with me. When I zoomed out, the orange spaCy points and the pink LLM points were spread across the United States, Europe, and even Japan and Korea, for a novel whose road never leaves the stretch between New York and New Orleans. About 27 percent of the mapped points in each automated layer fall outside the continental United States, and the LLM did no better than spaCy.

This post asks two questions. **What does a human reader use to place a name on a map that the machines do not? And what do the machines' wrong places reveal about the assumptions built into them?**

## Background: 1 the Road

*1 the Road* (Jean Boîte Éditions, 2018) was written by a car. In March 2017, Ross Goodwin drove a Cadillac from New York to New Orleans in the footsteps of Kerouac's *On the Road*, with a camera, a microphone, a GPS unit and a clock feeding a neural network (an LSTM) whose sentences were printed on receipt paper. This is why almost every page carries timestamps, coordinates, and lines such as "Waffle House: a restaurant in La Grange."

Those lines come from the Foursquare API. Kenric McDowell explains in his introduction that the AI knew nearby businesses because Foursquare exposed them, "by the priorities of the techno-capitalist producer and subject," which is why they were so often gas stations and fast food restaurants (p. 28). The novel's geography is filtered before anyone maps it: it is the American South seen through a check in app's database of businesses.

## Process

### Manual annotation in Recogito

Our assigned section was **Section 7**, the last leg of the trip along the Mississippi Gulf Coast, from Biloxi to New Orleans. I read the section and identified the words that referred to places, and Nahla researched each one and entered it in Recogito, either by linking it to a gazetteer entry or, when there was no match, by typing coordinates into the comment field.

<img src="{{ '/assets/images/a2_recogito_section7.png' | relative_url }}" style="zoom:50%;" />

*Figure 1. Section 7 in Recogito, with place names highlighted.*

<img src="{{ '/assets/images/a2_recogito_comment_coords.png' | relative_url }}" style="zoom:50%;" />

*Figure 2. Biloxi High School Stadium had no gazetteer match, so its coordinates were typed into the comment field.*

The hardest moments were names that looked like Biloxi but were not, such as "Biloxiana Parker" and "Biloxiski." We found nothing on Google Maps and concluded that the AI had invented them.

### Downloading and cleaning

I downloaded the annotations as CSV on **10 November 2026, around 23:17 (Gulf Standard Time)**. The export held all 250 annotations from the class, so I used the character offsets in the `ANCHOR` column to keep the 41 between the SECTION 7 and SECTION 8 headings.

| Cleaning step | Rows | Reason |
|---|---|---|
| Removed annotations of a lone period "." | 14 | Not place names (see Findings) |
| Relabelled "School" as Biloxi High School Stadium | 7 | Only the word "School" inside the repeated stadium name had been tagged, and it was linked to an elementary school in New Hampshire |
| Moved coordinates from `COMMENTS` to `LAT` / `LNG` | 11 | Places located by hand |
| Kept GeoNames coordinates | 8 | Biloxi, Gulfport, Bay Saint Louis, Slidell, New Orleans |
| Left without coordinates | 1 | Biloxiana Parker, marked "not identifiable" |

*Table 1. Cleaning of the Section 7 data: 41 rows in, 27 rows out.*

I relabelled the "School" rows rather than deleting them because the stadium really does recur, and both automated datasets count every recurrence. The cleaned file, with a `CLEANING_NOTE` on every row, and the automated datasets are in [this folder of my repository](https://github.com/jy3614/dhs/tree/master/assets/assignment2).

### Three datasets

| Layer | Who picked the place names | Who found the coordinates | Rows | Mapped | No coordinates |
|---|---|---|---|---|---|
| Manual (Recogito), Section 7 | Nahla and I | Gazetteer or research by hand | 27 | 26 | 1 |
| spaCy + geopy, pp. 41 to 140 | spaCy NER plus pattern rules | Nominatim (OpenStreetMap) | 341 | 248 | 93 |
| LLM, pp. 41 to 140 | An LLM, while transcribing the scans | Nominatim (OpenStreetMap) | 374 | 281 | 93 |

*Table 2. The three datasets. Both automated sets use the same LLM transcription and the same geocoder, so they differ only in how place names were picked out.*

### Making the map

I built the map in the posit.cloud notebook "Interactive Leaflet Map with Three Clickable Layers, 1 the Road." I pointed the manual layer to my cleaned file, fixed the basemap switcher (it listed a "Light" basemap that did not exist, so the satellite view could not be turned off), and replaced the default colors with the colorblind safe Okabe Ito palette.

The markers shrink from manual to spaCy to LLM and are drawn largest first, so agreement appears as a blue ring with an orange and a pink dot inside. The notebook also jitters each point by about 1 km; without it the pink points would hide the orange ones, since the automated methods gave identical coordinates for 163 names. The cost is that perfect agreement looks slightly off, but at world scale the jitter is invisible, so the overseas clusters are real.

<img src="{{ '/assets/images/a2_notebook_counts.png' | relative_url }}" style="zoom:50%;" />

*Figure 3. The notebook reports how many rows in each dataset have usable coordinates.*

## The Map

Use the checkboxes in the top right corner to toggle each layer or switch to the satellite view. Clicking a point shows its name, original coordinates and source fields.

<div style="width:100%; height:70vh;">
  <iframe
    src="{{ '/assets/assignment2/interactive_map.html' | relative_url }}"
    style="width:100%; height:100%; border:0;"
    loading="lazy">
  </iframe>
</div>

*Figure 4. Manual (blue), spaCy + geopy (orange) and LLM (pink) geocoding of 1 the Road.*

## Findings

### The road leaves the country

<img src="{{ '/assets/images/a2_map_world.png' | relative_url }}" style="zoom:50%;" />

*Figure 5. Zoomed out, the automated layers place points in Europe, South America, Africa and East Asia.*

The points that left the United States fall into a few kinds of names: town names shared with Europe (Florence to Italy, Augusta to Germany, Woodbridge and Winchester to England), chains (Chevron to London, Chipotle to Paris), road numbers (Exit 18 to Sapporo), generic nouns (Town Hall to Spain), and names the AI had bent ("Biloxie" to California). As a Korean, I was surprised to find that spaCy had put the word "School" in Busan.

The errors are also perfectly consistent. In the LLM layer, Florence appears 11 times and lands in Italy all 11 times. A machine that is wrong once is wrong every time the name returns.

### The LLM was not smarter than spaCy

For the 163 names that both methods picked, the coordinates are not just similar but identical, because both scripts hand the name to the same geocoder. Whatever intelligence the LLM has stops at recognizing the name; locating it is left to the same search engine.

The LLM's real advantage was in recognition. At Bay Saint Louis, the manual and LLM points fall 0.3 km apart, but there is no spaCy point, because spaCy cut the phrase as "Bay Saint Louis Stevensons" and the geocoder found nothing. The LLM also joined words broken across lines in the scan. These gains happen before the map, not on it.

<img src="{{ '/assets/images/a2_popup_manual.png' | relative_url }}" style="zoom:50%;" />
<img src="{{ '/assets/images/a2_popup_llm.png' | relative_url }}" style="zoom:50%;" />

*Figure 6. Bay Saint Louis: the manual and LLM points agree within 0.3 km. The spaCy layer has no point here.*

### We read the route, not the name

Unlike Assignment 1, I had no local knowledge here; I had never heard of Biloxi or Gulfport. What Nahla and I had was the story, read in order. The car does not teleport, so the next place must be near the last one. When the stadium has "a window on the Town Hall below," the town hall is Biloxi's, not Spain's. "Loraine: a neighborhood in Gulfport" is not Loraine, Illinois. "Biloxie," right after the stadium, is Biloxi with a typo.

The distances between our points and the LLM's show the two ways of reading. The gap is either almost nothing (Gulfport 0 km, Bay Saint Louis 0.3 km) or enormous (Loraine 1,102 km, Biloxie 2,801 km, Town Hall 7,528 km), with little in between. When the machines are right they are precise; when they are wrong they are on another continent, because each name is searched alone, without the route.

### Humans make mistakes too, but differently

The whole class's annotations changed my assumption that manual work is simply the most accurate. Groups recorded coordinates in different places (gazetteer links, comments, tags, even a Google Maps link), disagreed about what counts as a place, and made real mistakes: one group linked Camden to New Jersey instead of South Carolina. Our section produced the 14 "." annotations, all sharing one coordinate in the sea off Biloxi. Most sit inside OCR junk, where the scanner turned images into symbols, and Recogito's option to apply one annotation to every matching string copied them across the text.

The pattern is the interesting part. Human errors were scattered, different in every group, and stayed on the route; even Camden, New Jersey is near where the trip began. They became repetitive only when a decision was handed to the tool. In the Camden case the LLM made exactly the same mistake, most likely because the group accepted Recogito's first suggestion. The human stopped reading the route, trusted the ranking, and became as wrong as the machine.

## Critical Discussion

McDowell's point about Foursquare explains much of what the machines got wrong. In Foursquare, a gas station arrives with its name and coordinates together, because the car is really passing it; as volunteered geographic information, it shows only what users and businesses chose to register. The novel kept the name and dropped the coordinates, and geocoding tries to rebuild them from the name alone. For a chain like BP, which exists in thousands of places, this cannot work: the geocoder returns the same spot in Minnesota all 8 times. The format survives even when the content does not. In "Biloxi High School Stadium: a football stadium in Biloxiana Parker," the slot for the town is intact, but the town is an invention.

The wrong places also reveal assumptions nobody wrote down. The geocoder takes exactly one answer, searches the whole world, ranks famous places first (Florence, Italy over Florence, South Carolina), and searches each name in isolation; spaCy assumes a capitalized word is a name. Restricting the search to the United States would probably have removed most overseas points. The errors come less from what the machines cannot do than from the assumption that the novel could be anywhere.

This shows the limits of computational approaches to spatial data. Every method, ours included, treats a place as a single point, which a highway or a state line is not. The machines cannot say "this place does not exist," so an invented Biloxiana Parker looks the same as a real place they failed to find. And the map rests on layers of automation (Foursquare, the LSTM, OCR, the LLM transcription, NER, the geocoder), each passing its mistakes on.

Finally, the map shows only what survived editing. McDowell writes that the project generated 200,000 words (p. 18), while the printed novel (pp. 41 to 141) has roughly 16,000 to 17,000, about one word in twelve. The map cannot show what was cut, but repetitions may hint at it: Biloxi High School Stadium appears 11 times, which suggests the car was standing still. The map may record less the road than the places where the car stopped.

## Conclusion

I began this project expecting a simple ranking, with humans on top and the LLM ahead of spaCy. Instead, what separates the methods is not intelligence but context. Nahla and I placed names by following the route, reading through typos, and deciding when a place did not exist. The machines placed each name alone, assuming one correct location somewhere in the world, and repeated their mistakes with perfect consistency. Humans were wrong too, most clearly when we trusted the tool instead of the story. From now on, when I see a map produced automatically from text, I will ask not only whether the points are right, but what the method assumed about where the text could be.

## Division of Work

I read Section 7 and identified the place names; my classmate Nahla researched and entered each place and its coordinates in Recogito. I downloaded and cleaned the data, built the map, and wrote this post.

## References

- Goodwin, Ross. *1 the Road*. Paris: Jean Boîte Éditions, 2018.
- McDowell, Kenric. "AI Poetry Hits the Road." In Goodwin, *1 the Road*, pp. 18 to 37. Paris: Jean Boîte Éditions, 2018.
- Recogito. Pelagios Network. https://recogito.pelagios.org
- Wrisley, David Joseph. "Interactive Leaflet Map with Three Clickable Layers, 1 the Road." posit.cloud notebook, 2026.
- spaCy (en_core_web_sm) and geopy with the Nominatim geocoder (OpenStreetMap).

## Generative AI Statement

