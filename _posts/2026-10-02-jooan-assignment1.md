---
title: "Assignment 1: Mountains, Hills and Peaks of South Korea in GeoNames"
permalink: /assignment1/
last_modified_at: 2026-10-02T22:00:00+04:00
tags:
  - GeoNames
  - Interactive Map
  - R
---

## Introduction

I grew up in South Korea for twenty years, and I spent a lot of that time hiking. For me, mountains are not just one feature of Korean geography; they are its defining character. So when I opened the GeoNames file for South Korea, I went straight to its terrain, and I found a puzzle. In everyday Korean, almost every raised landform is simply called *san* (산), a mountain. GeoNames, however, splits this single Korean category into three feature codes: **HLL** (hill), **MT** (mountain), and **PK** (peak).

This post asks one question: **by what criteria does GeoNames divide what Koreans call "mountains" into hills, mountains, and peaks, and what does that division do to the picture of Korea it produces?**

## Background and Expectations

Before starting, there are some things that I want to introduce about my country. South Korea is commonly described as a country where about 70 percent of the land is mountainous. I also knew how Koreans use the word for hill, *eondeok* (언덕). It usually means a small slope, the kind of rise behind a school that children run up. Even a very small mountain is almost never called a hill. In the official classifications I grew up with, I learned about mountains, but I never heard anyone count how many hills Korea has. Nobody had a reason to ask, because "hill" is not really a category in the Korean way of seeing land.

My expectations were therefore mixed. I assumed GeoNames would follow a foreign, most likely English language, classification system, which meant I honestly could not predict how many hills or mountains it would contain. Still, perhaps because "mountain" is the category I am used to, I expected MT to be far larger than HLL.

The dataset itself was smaller than I expected. The KR file contains **144,733 rows**, covering every kind of feature from villages to bridges. Korea has about 50 million people, roughly half of them living in the capital region, and a great deal of publicly available data. If all of its towns, schools, temples, landforms, and buildings add up to only about 145,000 named entries, I feel that is too underestimated.

| Feature class | Rows |
|---|---|
| P (cities, villages) | 62,751 |
| S (spots, buildings) | 37,243 |
| T (terrain) | 15,934 |
| H (streams, lakes) | 12,271 |
| L (parks, areas) | 8,324 |
| A (administrative divisions) | 6,877 |
| R (roads, railroads) | 1,302 |
| V (vegetation) | 31 |

*Table 1. Rows in the South Korea file by GeoNames feature class.*

The most common feature code by far is PPL (populated place), with 59,326 rows, about 41 percent of the whole file. It is followed by RSV (reservoir, 8,689), SCH (school, 7,829), and BDG (bridge, 7,640). While many villages, schools, and bridges matched my expectations, reservoirs in second place quite surprised me because although I feel like Korea does have many reservoirs, I did not expect them to outnumber almost every other kind of place.

The update history also made me raise questions. Looking at the year each entry was last modified, **105,341 rows, about 73 percent of the file, were last modified in 2023**, while the rest are scattered between 1993 and 2026. Did one person, or one large import, change most of the country at once? Why 2023 in particular? I wondered whether growing demand for online geospatial data around the COVID period played a role, but I could not confirm this, so I leave it as an open question. And if most of the data was last touched three years ago, can I treat it as current? I could not fully answer these questions here, but they made me read everything that follows with some caution.

**Why HLL, MT, and PK?** I chose these three codes because together they let me test the classification itself. All three describe what Korean would simply call *san*, so comparing them shows how GeoNames cuts one Korean concept into three. They were also balanced enough to map: HLL has 2,742 points, MT has 2,658, and PK has 1,205.

## The Map

Use the checkboxes in the top right corner to turn each layer on and off. Clicking a point shows its name, feature code, and elevation.

<div style="width:100%; height:70vh;">
  <iframe
    src="{{ '/assets/maps/KR_featuremap.html' | relative_url }}"
    style="width:100%; height:100%; border:0;"
    loading="lazy">
  </iframe>
</div>

## Findings

### Hills outnumber mountains

The first result genuinely surprised me: **GeoNames lists more hills (2,742) than mountains (2,658) in South Korea.** HLL even appears among the ten most common feature codes for the entire country, while MT does not. This was the moment I felt, very concretely, that this dataset is not built on Korean categories. It also made me curious about the rule GeoNames uses to tell the two apart.

### Where each layer appears

<img src="{{ '/assets/images/hll_layer.png' | relative_url }}" style="zoom:50%;" />

*Figure 1. HLL layer only. Hills cluster in the capital region and along the west coast.*

In my experience, hills exist fairly evenly across the country. Here they concentrate around the capital region and the west coast. My interpretation is that this reflects where many people live, and perhaps where more data gets entered, rather than where hills actually are.

<img src="{{ '/assets/images/mt_layer.png' | relative_url }}" style="zoom:50%;" />

*Figure 2. MT layer only. Mountains are spread almost evenly across the country.*

This also puzzled me. Korean geography is often summarized as "high in the east, low in the west" (*donggo seojeo*, 동고서저), with the Taebaek range running along the east coast. If the MT layer followed the terrain, I would expect it to be denser in the east. The even spread suggests that the classification does not follow height alone.

<img src="{{ '/assets/images/pk_layer.png' | relative_url }}" style="zoom:50%;" />

*Figure 3. PK layer only. Peaks appear across the country, but there is a visible gap around Jirisan in the south.*

### Is elevation the rule?

| Code | Rows | Median elevation (DEM) | Highest elevation (DEM) |
|---|---|---|---|
| HLL | 2,742 | 133 m | 969 m |
| MT | 2,658 | 471 m | 1,883 m |
| PK | 1,205 | 376 m | 1,714 m |

*Table 2. Elevation by feature code, using the approximate elevation from a digital elevation model (the `dem` column in GeoNames).*

At first glance, elevation seems to explain the split. The median hill is 133 m and the median mountain is 471 m, so hills are generally smaller, mountains generally higher, and peaks are the tops of higher mountains. But the ranges overlap heavily: **the highest "hill" (969 m) is more than twice as high as the median "mountain" (471 m).** Elevation is clearly part of the story, but it is not a consistent rule.

My own neighborhood confirmed this. In Goyang, Hwangnyongsan is a lower mountain than Gobongsan, yet GeoNames classifies Hwangnyongsan as MT and Gobongsan as HLL. (Hwangnyongsan is 135m high, while Gobongsan is 208m high) This is one example I could recognize because I know these mountains personally, and I do not think Goyang is a single strange exception because of the overlap in Table 2.

### The highest peak is not a peak

Another striking finding came from Jirisan. Its summit, Cheonwangbong, is the highest peak on the Korean mainland at 1,915 m. When I searched the data for it, I first found a "Cheonwangbong" coded PK with only 556 m of elevation, and thought GeoNames had recorded the wrong elevation. When I checked the coordinates just in case, however, that point turned out to be around 100 km away from the actual peak. It turned out that it was a different place, and the actual "Cheonwangbong" that I was looking for was nowhere on the map. In other words, **the highest peak on the Korean mainland is not classified as a peak at all.** It was also a lesson for me: relying on names alone, I almost reported the wrong error.

## Critical Discussion

Kitchin and Lauriault, drawing on Ian Hacking's idea of the "looping effect," describe classification as grouping together things that are believed to share characteristics, and note that things which do not fit are sometimes pushed into a group (p. 11), and I found that this describes my findings closely. In Korean, *san* is one category. GeoNames forces it into three classes, with results that do not fit cleanly. A lower mountain in Goyang becomes MT while a taller one becomes HLL, and the highest summit on the mainland is not counted as a peak. Someone who knew Korea only through this data could reasonably conclude that it is a land of hills more than of mountains, which is not at all how Koreans experience their country.

The *Do Maps Lie?* video makes a similar point with house prices in London. Jim and Anna use exactly the same data, but by choosing different classes (equal intervals in one map, above or below the national average in the other), they tell opposite stories. GeoNames does something similar to Korean terrain. The landforms are real, but the classes of hill, mountain, and peak decide what the map appears to say. The video ends by asking three questions of any map, and applying them to GeoNames is revealing:

- **Who made it?** Contributors from around the world, including a listed GeoNames ambassador for South Korea, Sangchul Ahn. I was not able to trace where the original Korean data came from, which remains an open question for this project.
- **Why was it made?** To record places all over the world within one shared, English language system of categories.
- **What does it actually tell us?** Where named points are located, not the shape or extent of the land.

The existence of a Korean ambassador makes the Jirisan result more interesting, not less. Having a local contact does not guarantee that local categories shape the data, or that the most important features are classified in the way locals would expect.

## Transferability

From now on, when I look at geospatial data, I will try not to simply accept what is presented on the map, but instead, I want to think carefully about who made it, why it was made, and what it actually tells me, and to receive that information more critically.

## Conclusion

I began this project expecting GeoNames to confirm what I already knew: that Korea is a country of mountains. It did not, but not because Korea has fewer mountains than I thought. It is because the dataset sorts Korean landforms into categories the Korean language does not use, applies those categories inconsistently, and turns vast mountain ranges into single points.

## References

- GeoNames. South Korea country file (KR.txt) and feature codes. https://www.geonames.org
- Kitchin, Rob, and Tracey P. Lauriault. "Toward Critical Data Studies: Charting and Unpacking Data Assemblages and Their Work." In *Thinking Big Data in Geography*, edited by Jim Thatcher, Josef Eckert, and Andrew Shears, pages 3 to 20. University of Nebraska Press, 2018.
- "Do Maps Lie?" YouTube video. https://www.youtube.com/watch?v=G0_MBrJnRq0

## Generative AI Statement

I used Claude for the necessary part of this assignment. It helped me troubleshoot errors on GitHub pages, and suggested console commands for counting and summarizing rows. It helped me brainstorm when it comes to narrowing down the feature codes of my interest. The observations, interpretations, local knowledge (such as the Goyang and Jirisan cases), and the final choice of feature codes are my own.
