---
title: "Assignment 2: Manual vs. Automated Geocoding of 1 the Road"
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

Before I opened the map, I had a definite ranking in mind about the accuracy of each method. The manual annotations that my classmate and I made in Recogito would be the most accurate, and of the two automated methods, the LLM would be better than spaCy, because spaCy reads words exactly as they are written while an LLM can read, understand, and reflect the context.

However, the map was somewhat different from my expectation. When I zoomed out, the orange spaCy points and the pink LLM points were spread across the United States, Europe, and even Korea and Japan, for a novel whose road never leaves the stretch between New York and New Orleans. About 27 percent of the mapped points in each automated layer fall outside the continental United States, and the LLM did no better than spaCy.

This post asks two questions. **What does a human reader use to place a name on a map that the machines do not? And what do the machines' wrong places reveal about the assumptions built into them?**

## Background: 1 the Road

*1 the Road* (Jean Boîte Éditions, 2018) was written by a car. In March 2017, Ross Goodwin drove a Cadillac from New York to New Orleans in the footsteps of Kerouac's *On the Road*, with a camera, a microphone, a GPS unit and a clock feeding a neural network (an LSTM) whose sentences were printed on receipt paper. This is why almost every page carries timestamps, coordinates, and lines such as "Waffle House: a restaurant in La Grange."

Those lines come from the Foursquare API. Kenric McDowell explains in his introduction that the AI knew nearby businesses because Foursquare exposed them, "by the priorities of the techno-capitalist producer and subject," which is why they were so often gas stations and fast food restaurants (p. 28). The novel's geography is filtered before anyone maps it: the places it “sees” are the businesses that people registered on Foursquare, not the landmarks a traveler might notice.

## Process

### Manual annotation in Recogito

Our assigned section was **Section 7**, the last leg of the trip along the Mississippi Gulf Coast, from Biloxi to New Orleans. I read the section and identified the words that referred to places, and my classmate (Nahla) researched each one and entered it in Recogito, either by linking it to a gazetteer entry or, when there was no match, by typing coordinates into the comment field.

<img src="{{ '/assets/images/a2_recogito_section7.png' | relative_url }}" style="zoom:50%;" />

*Figure 1. Section 7 in Recogito, with place names highlighted.*

<img src="{{ '/assets/images/a2_recogito_comment_coords.png' | relative_url }}" style="zoom:50%;" />

*Figure 2. Biloxi High School Stadium had no gazetteer match, so its coordinates were typed into the comment field.*

The hardest moments were names that looked like Biloxi but were not, such as "Biloxiana Parker" and "Biloxiski." We found nothing on Google Maps and concluded that the AI had invented them.

### Downloading and cleaning

I downloaded the annotations as CSV, and because the export held all 250 annotations from the class, The export held all 250 annotations from the class, with no column showing who made them. Each annotation, however, records its position in the text as a character number, so I kept only the 41 that fall between the SECTION 7 and SECTION 8 headings.

| Cleaning step | Rows | Reason |
|---|---|---|
| Removed annotations of a lone period "." | 14 | Punctuation, not places |
| Relabelled "School" as Biloxi High School Stadium | 7 | Only the word "School" inside the repeated stadium name had been tagged, and it was linked to an elementary school in New Hampshire |
| Moved coordinates from `COMMENTS` to `LAT` / `LNG` | 11 | Places located by hand |
| Kept GeoNames coordinates | 8 | Biloxi, Gulfport, Bay Saint Louis, Slidell, New Orleans |
| Left without coordinates | 1 | Biloxiana Parker, marked "not identifiable" |

*Table 1. Cleaning of the Section 7 data: started with 41 annotations and kept 27.*

I relabelled the "School" rows rather than deleting them because the stadium really does recur, and both automated datasets count every recurrence. The cleaned file, with a `CLEANING_NOTE` on every row, and the automated datasets are in [this folder of my repository](https://github.com/jy3614/dhs/tree/master/assets/assignment2).

### Three datasets

| Layer | Who picked the place names | Who found the coordinates | Rows | Mapped | No coordinates |
|---|---|---|---|---|---|
| Manual (Recogito), Section 7 | my classmate and I | Gazetteer or research by hand | 27 | 26 | 1 |
| spaCy + geopy, pp. 41 to 140 | spaCy NER plus pattern rules | Nominatim (OpenStreetMap) | 341 | 248 | 93 |
| LLM, pp. 41 to 140 | An LLM, while transcribing the scans | Nominatim (OpenStreetMap) | 374 | 281 | 93 |

*Table 2. The three datasets. Both automated sets use the same LLM transcription and the same geocoder, so they differ only in how place names were picked out.*

### Making the map

I built the map in the posit.cloud notebook "Interactive Leaflet Map with Three Clickable Layers, 1 the Road." I pointed the manual layer to my cleaned file, fixed the basemap switcher (it listed a “Light” basemap that was never created and left out the satellite layer, so switching basemaps would not have worked), and replaced the default colors with colors that are easier to distinguish.

The manual points are the largest and the LLM points the smallest, and the larger ones are drawn first, so overlapping points do not hide one another. The notebook also shifts every point randomly by about 1 km (“jitter”). Without it, the pink LLM points would sit exactly on top of the orange spaCy points wherever the two methods agree, which they do for 163 names. There is one cost, however, which is that identical locations look slightly apart when zoomed in.

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

The LLM's real advantage was in recognition. At Bay Saint Louis, the manual and LLM points fall 0.3 km apart, but there is no spaCy point, because spaCy cut the phrase as "Bay Saint Louis Stevensons" and the geocoder found nothing. The LLM also joined words broken across lines in the scan. 

<img src="{{ '/assets/images/a2_popup_manual.png' | relative_url }}" style="zoom:50%;" />
<img src="{{ '/assets/images/a2_popup_llm.png' | relative_url }}" style="zoom:50%;" />

*Figure 6. Bay Saint Louis: the manual and LLM points agree within 0.3 km. The spaCy layer has no point here.*

### Humans read the route, not the name

Unlike Assignment 1, I had no local knowledge here; I had never heard of Biloxi or Gulfport. What my classmate and I had was the story, read in order. The car does not teleport, so the next place must be near the last one. When the stadium has "a window on the Town Hall below," the town hall is Biloxi's, not Spain's. "Loraine: a neighborhood in Gulfport" is not Loraine, Illinois. "Biloxie," right after the stadium, is Biloxi with a typo.

The distances between our points and the LLM's show the two ways of reading. The gap is either almost nothing (Gulfport 0 km, Bay Saint Louis 0.3 km) or enormous (Loraine 1,102 km, Biloxie 2,801 km, Town Hall 7,528 km), with little in between. When the machines are right they are precise; when they are wrong they are on another continent, because each name is searched alone, without the route.

## Critical Discussion

Part of the reason the machines failed lies in where the data came from. McDowell explains that the novel’s places were supplied by the Foursquare API, which is why most of them are gas stations and fast food restaurants (p. 28). Foursquare gave the car each place’s name and coordinates, but the novel kept only the name. Geocoding cannot recover what was lost, especially for chains: BP was placed at the same spot in Minnesota all 8 times.

These errors also answer my second question about what the machines assume. The geocoder assumes every name has one answer, searches the whole world, and puts famous places first, which is why Florence landed in Italy. Limiting the search to the United States would likely have removed most overseas points. The machines did not fail to find these places; they simply did not know where the novel was.

Finally, the map has a limit of its own: it shows only what survived editing. The project produced 200,000 words (p. 18), but the printed novel has about 16,000 to 17,000, roughly one in twelve. Any map of 1 the Road is a map of a small, edited fraction of what the car wrote.

## Conclusion

I began this project expecting a clear order of accuracy, with humans on top and the LLM ahead of spaCy. Instead, what separates the methods is not intelligence but context. My classmate and I placed names by following the route, reading through typos, and deciding when a place did not exist. The machines placed each name alone, assuming one correct location somewhere in the world, and repeated their mistakes with consistency. Our own data had errors too, but they were scattered and stayed on the route. Therefore, next time I use an automated tool to locate places, I will give it the context it lacks, such as the country or state, and check a few results by hand before trusting the rest.

## Division of Work

I read Section 7 and identified the place names; my classmate researched and entered each place and its coordinates in Recogito. After that, for the Assignment itself, I worked individually; I downloaded and cleaned the data, built the map, and wrote this post.

## References

- Goodwin, Ross. *1 the Road*. Paris: Jean Boîte Éditions, 2018.
- McDowell, Kenric. "AI Poetry Hits the Road." In Goodwin, *1 the Road*, pp. 18 to 37. Paris: Jean Boîte Éditions, 2018.

## Generative AI Statement

I used Claude for the necessary part of this assignment. It helped me filter the class Recogito export to Section 7, move the coordinates from the comment field into the latitude and longitude fields, and fix the basemap settings in the notebook. The annotation work and the interpretations are my own.

**READY FOR GRADING**
