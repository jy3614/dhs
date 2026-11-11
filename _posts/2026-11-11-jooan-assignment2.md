---
title: "Assignment 2: Manual vs. Automated Geocoding of 1 the Road"
permalink: /assignment2/
last_modified_at: 2026-11-11T23:00:00+04:00
tags:
  - Recogito
  - Geocoding
  - NER
  - Interactive Map
  - R
---

## Introduction

✏️ TODO: 2 to 3 sentences. What is this post about, and what one question will it answer? (Example direction: what do a human reader and two automated pipelines each "see" as a place in an AI written novel, and why do they disagree?)

## Background: 1 the Road

✏️ TODO: Who is Ross Goodwin, how was the novel produced (the 2017 drive from New York to New Orleans, the camera, microphone, GPS and clock feeding an LSTM, Foursquare data), and how was it published (Jean Boîte Éditions, 2018).

✏️ TODO: McDowell's introduction, p. 28, on the Foursquare sourced locations (gas stations, fast food restaurants). One short quote or paraphrase, and why it matters for geocoding.

## Process

### Step 1: Manual annotation in Recogito

Our assigned section was **Section 7**, which covers the last leg of the trip along the Mississippi Gulf Coast, from Biloxi to New Orleans. My partner Nahla Omran and I divided the work in two stages: I read through the section and identified the words that referred to places, and Nahla researched each place and entered it in Recogito, either by linking it to a gazetteer entry or by typing coordinates into the comment field when no match existed.

<img src="{{ '/assets/images/a2_recogito_section7.png' | relative_url }}" style="zoom:50%;" />

*Figure 1. Section 7 in Recogito, with place names highlighted.*

<img src="{{ '/assets/images/a2_recogito_comment_coords.png' | relative_url }}" style="zoom:50%;" />

*Figure 2. Biloxi High School Stadium had no gazetteer match, so its coordinates were typed into the comment field.*

✏️ TODO: What worked and what did not while annotating? (Example: names the AI distorted, like "Biloxie" or "Biloxiana Parker".)

### Step 2: Downloading and cleaning

I downloaded the annotations as CSV using Recogito's download options on **10 November 2026, around 23:17 (Gulf Standard Time)**. The export contained 250 annotations from the whole class, with no column identifying the annotator, so I used the character offset in the `ANCHOR` column to keep only the 41 annotations that fall between the SECTION 7 and SECTION 8 headings.

| Cleaning step | Rows | Reason |
|---|---|---|
| Removed annotations of a lone period "." | 14 | Punctuation or OCR noise, apparently created by applying one annotation to every matching string |
| Relabelled "School" as Biloxi High School Stadium | 7 | Only the word "School" inside the repeated stadium name had been tagged, and it was linked to Allenstown Elementary School in New Hampshire |
| Moved coordinates from `COMMENTS` to `LAT` / `LNG` | 11 | Places with no gazetteer match, located by hand |
| Kept GeoNames coordinates | 8 | Biloxi, Gulfport, Bay Saint Louis, Slidell, New Orleans |
| Left without coordinates | 1 | Biloxiana Parker, marked "not identifiable" |
| Filled the empty `TYPE` field | all | PLACE for names, GEO for coordinate strings, to match the two automated files |

*Table 1. Cleaning of the Section 7 data: 41 rows in, 27 rows out.*

The cleaned file and the two automated datasets are in [this folder of my repository](https://github.com/jy3614/dhs/tree/master/assets/assignment2). Each row of the cleaned file has a `CLEANING_NOTE` explaining what was changed.

✏️ TODO: One or two sentences on the "School" decision in your own words (why relabel rather than delete).

### Step 3: Three datasets

| Layer | How places were found | How coordinates were found | Coverage | Rows | Mappable | No coordinates |
|---|---|---|---|---|---|---|
| Manual (Recogito) | Read by us | Gazetteer or researched by hand | Section 7 | 27 | 26 | 1 |
| spaCy + geopy | spaCy NER (GPE, LOC, FAC) plus pattern rules | Nominatim via geopy | pp. 41 to 140 | 341 | 248 | 93 |
| LLM | Tagged by an LLM while transcribing the scans to TEI | Nominatim via geopy | pp. 41 to 140 | 374 | 281 | 93 |

*Table 2. The three datasets compared in this post.*

✏️ TODO: Point out that both automated files start from the same LLM transcription and use the same geocoder, so the difference between them comes only from how place names were picked out.

### Step 4: Making the map

I built the map in the posit.cloud notebook "Interactive Leaflet Map with Three Clickable Layers, 1 the Road." I changed the file path of the manual layer to my cleaned CSV, corrected the basemap switcher (it listed a "Light" basemap that did not exist, so the satellite layer could not be turned off), and replaced the default colors with the colorblind safe Okabe Ito palette. The notebook jitters each point by about 0.01 degrees (roughly 1 km) so that overlapping points stay visible; the popups still report the original coordinates.

<img src="{{ '/assets/images/a2_notebook_counts.png' | relative_url }}" style="zoom:50%;" />

*Figure 3. The notebook's report of rows with and without usable coordinates in each dataset.*

## The Map

Use the checkboxes in the top right corner to turn each layer on and off, and switch between the street map and satellite view. Clicking a point shows the place name, its original coordinates and the other fields from its source file.

<div style="width:100%; height:70vh;">
  <iframe
    src="{{ '/assets/assignment2/interactive_map.html' | relative_url }}"
    style="width:100%; height:100%; border:0;"
    loading="lazy">
  </iframe>
</div>

*Figure 4. Manual (blue), spaCy + geopy (orange) and LLM (pink) geocoding of 1 the Road.*

## Findings

<img src="{{ '/assets/images/a2_map_world.png' | relative_url }}" style="zoom:50%;" />

*Figure 5. Zoomed out, the automated layers place points in Europe, South America, Africa and East Asia.*

✏️ TODO: Finding 1, places that left the country. Why so many in Britain and Europe? (Woodbridge, Winchester, Goldsborough, Florence, Augusta...)

<img src="{{ '/assets/images/a2_popup_manual.png' | relative_url }}" style="zoom:50%;" />
<img src="{{ '/assets/images/a2_popup_llm.png' | relative_url }}" style="zoom:50%;" />

*Figure 6. Bay Saint Louis: the manual and LLM points fall about 200 m apart. The spaCy layer has no point here.*

✏️ TODO: Finding 2, agreement and boundaries. Bay Saint Louis agrees for manual and LLM, but spaCy cut the name as "Bay Saint Louis Stevensons" and found nothing.

✏️ TODO: Finding 3, misspellings and invented names. "Biloxie" (Biloxi to a human, California to Nominatim), "Biloxiana Parker" (no one could place it).

✏️ TODO: Finding 4, the 93 unmatched places in each automated file, and the 1 in ours. What does that number say?

✏️ TODO: Finding 5 (optional), human errors too: the "School" and "." annotations show that manual work has its own tool driven mistakes.

## Critical Discussion

✏️ TODO: Drucker's chapter on mapping. One idea and how it fits these maps.

✏️ TODO: What humans do that the machines cannot (context of the route, knowing the trip ends in New Orleans), and what assumptions spaCy and geopy make (capitalized words are names; the first or most prominent match worldwide is the right one).

✏️ TODO: The limits of computational spatial data, and what the novel's length suggests about how much the editor trimmed.

## Division of Work

I read Section 7 and identified the place names; Nahla Omran researched and entered each place and its coordinates in Recogito. I downloaded and cleaned the data, built the map and wrote this post.

✏️ TODO: Check this with Nahla and adjust.

## Conclusion

✏️ TODO: 3 to 4 sentences.

## References

- Goodwin, Ross. *1 the Road*. Paris: Jean Boîte Éditions, 2018.
- McDowell, Kenric. Introduction to *1 the Road*, 2018.
- Drucker, Johanna. ✏️ TODO: full citation of the mapping chapter.
- Recogito. Pelagios Network. https://recogito.pelagios.org
- Wrisley, David Joseph. "Interactive Leaflet Map with Three Clickable Layers, 1 the Road." posit.cloud notebook, 2026.

## Generative AI Statement

I used Claude for parts of this assignment. It filtered the class wide Recogito export to Section 7 using the character offsets, moved the coordinates from the comment field into the latitude and longitude fields, and flagged the "." and "School" annotations; the decision to relabel the "School" rows was mine. It also identified which files belonged to which automated approach, pointed out the file path and basemap lines to change in the notebook, and helped me format this post. ✏️ TODO: adjust to what you actually used it for, and add that the observations and interpretations are your own.

✏️ TODO: When everything above is finished, delete every ✏️ line and add READY FOR GRADING in bold here.
