---
title: 'An analysis of sex ratios using a biodiversity data cube'
tags:
  - ducks
  - biodiversity indicators
  - Anatidae
  - Europe
  - mapping
  - GBIF
authors:
  - name: Quentin Groom
    orcid: 0000-0002-0596-5376
    affiliation: 1
  - name: Maarten Trekels
    orcid: 0000-0001-8282-8765
    affiliation: 1

affiliations:
 - name: Meise Botanic Garden, Nieuwelaan 38, 1860 Meise, Belgium
   index: 1
date: 05. April 2024
bibliography: paper.bib
authors_short: Groom, Q.
group: Project 12
event: Hacking Biodiversity Data Cubes for Policy, Brussels, Belgium, 2024
biohackathon_name: Hacking Biodiversity Data Cubes for Policy
biohackathon_url: https://b-cubed.eu/b-cubed-hackathon
biohackathon_location: Brussels, Belgium
---

## Abstract
This investigation uses biodiversity data cubes derived from the datasets mobilised by the Global Biodiversity Information Facility (GBIF), to conduct an analysis of sex ratios of ducks across Europe. Encompassing over 4 million occurrences extracted from nearly 5000 datasets, this study elucidates sex distribution patterns across various species, focussing on temporal and spatial dynamics. The aim of this study is to highlight the availability of open sex data and its potential usefulness in research and monitoring of sex ratios of wild organisms, particularly in sexual dimorphic species.

## Introduction
The balance of sex ratios in animal populations is a critical indicator of ecological health and evolutionary dynamics [@10.1134/S2079086421030099]. Alterations in these ratios can reflect environmental stresses, reproductive strategies, or human influence. Biodiversity data cubes, particularly those derived from the diverse data mobilised by GBIF, provide a pathway to investigate various biological phenomena [@10.1101/2020.03.23.983601]. These data may originate from diverse sources, including citizen science projects, ringing activities, breeding bird counts, and hunting bag records. This variety introduces a degree of heterogeneity not typically encountered in more specialised surveys, yet it potentially enables comprehensive analyses across broader temporal and spatial scales.

This study leverages such a data cube to examine sex ratio dynamics within the European ducks, swans and geese, aiming to uncover patterns that could inform conservation efforts and ecological understanding. However, the results presented here are only intended to draw attention to the availability of such data and the use of data cubes to analyse them. Therefore, the analysis is to give readers an overview of the characteristics of the data that are available.

The family of ducks, geese, and swans (Anatidae) was chosen as an exemplar group, because it contains species that are both sexually monomorphic (e.g. *Branta canadensis*) and dimorphic (e.g. *Somateria mollissima*). Sex is easy to determine, even at a distance, for a human observer in sexually dimorphic species, whereas sexually monomorphic Anatidae would usually have to be captured to determine their sex reliably. There are many other examples of sexually dimorphic vertebrates that could be analysed from data mobilised from GBIF, including many other species of bird, deer, fish, primates and pinniped. There are also many sexually dimorphic insects, including some Odonata, Coleoptera and Lepidoptera. And, although plants are not strongly sexually dimorphic there are many dioecious species, which can be sexed easily when flowering, examples include members of the genera *Asparagus*, *Cycas*, *Diospyros*, *Ginkgo*, *Juniperus* and *Salix*.

Here we describe a Jupyter notebook written to analyse sex from a datacube. This script is openly licensed and could be repurposed to explore the availability of sex data on GBIF for any taxa. Though for a more serious examination of sex ratios of organisms the users may wish to write their own code and also examine the datasets that contributed to this data cube more critically.

## Materials and Methods
The data cube was generated from the SQL query run on the 2nd April 2024 detailed below. It extracted 4,038,527 aggregated rows from GBIF [@10.15468/dl.aqumsn]. The data are restricted to GBIF family key 2986 that is the code for the Anatidae, the  family that includes ducks, geese, and swans. The records were aggregated to a 10 km^2^ grid using the EEA grid system. Aggregated occurrences were also restricted to those after 1900 and to the continent of Europe. Records already identified within the GBIF infrastructure as invalid were excluded.
The resulting dataset aggregates data from 4,985 published datasets on GBIF, involving 230 publishers.
https://www.gbif.org/occurrence/download/0083528-240321170329656


> SELECT "year", gbif_eeargCode(10000, decimalLatitude, decimalLongitude, COALESCE(coordinateUncertaintyInMeters, 10000)) AS eeaCellCode, speciesKey, COUNT(*) AS 'count', SUM(CASE WHEN sex = 'FEMALE' THEN 1 ELSE 0 END) AS female_count, SUM(CASE WHEN sex = 'MALE' THEN 1 ELSE 0 END) AS male_count, SUM(CASE WHEN sex = 'HERMERMAPHRODITE' THEN 1 ELSE 0 END) AS hermaphrodite_count, MIN(COALESCE(coordinateUncertaintyInMeters, 10000)) AS minCoordinateUncertaintyInMeters FROM occurrence WHERE occurrenceStatus = 'PRESENT' AND familyKey = 2986 AND NOT array_contains(issue, 'ZERO_COORDINATE') AND NOT array_contains(issue, 'COORDINATE_OUT_OF_RANGE') AND NOT array_contains(issue, 'COORDINATE_INVALID') AND NOT array_contains(issue, 'COUNTRY_COORDINATE_MISMATCH') AND (identificationVerificationStatus IS NULL OR NOT ( LOWER(identificationVerificationStatus) LIKE '%unverified%' OR LOWER(identificationVerificationStatus) LIKE '%unvalidated%' OR LOWER(identificationVerificationStatus) LIKE '%not able to validate%' OR LOWER(identificationVerificationStatus) LIKE '%control could not be conclusive due to insufficient knowledge%' OR LOWER(identificationVerificationStatus) LIKE '%unconfirmed%' OR LOWER(identificationVerificationStatus) LIKE '%unconfirmed - not reviewed%' OR LOWER(identificationVerificationStatus) LIKE '%validation requested%' ) ) AND "year" >= 1900 AND continent = 'EUROPE' AND hasCoordinate GROUP BY "year", eeaCellCode, speciesKey ORDER BY "year" DESC, eeaCellCode ASC, speciesKey ASC;

Through the strategic exclusion of the `dwc:individualCount` from our analysis to simplify the initial approach, we aim to provide a foundational understanding of sex ratio variations and how they could be use to monitor population status and trends of biodiversity. However, a more focused study may wish to examine whether it is useful to consider `dwc:individualCount` in the aggregation step.

Shape files of the European borders were sources from Natural Earth. Free vector and raster map data @ naturalearthdata.com.

Our analytical framework is predicated on a selective extraction from the GBIF dataset, focusing on records designated as "PRESENT" while excluding data compromised by spatial inaccuracies. The analysis was facilitated by Python’s scientific stack, including pandas for data manipulation, Matplotlib and seaborn for visualisation, and GeoPandas with Shapely for spatial analysis. This study processes the derived biodiversity data cube, augmenting it with necessary spatial and temporal attributes, and preparing it for analysis. Kriging was conducted using PyKrige with a hole-effect variogram_model [@10.5281/zenodo.10016909]. Versions of Python packages are detailed in table 1.

Table 1. Versions of Python packages used

|   Package  | version |                  Website                  |
|----------|-------|-----------------------------------------|
| geopandas  | 0.14.3  | https://geopandas.org/en/stable/          |
| matplotlib | 3.7.1   | https://matplotlib.org/                   |
| numpy      | 1.24.3  | https://numpy.org/                        |
| pandas     | 2.0.3   | https://pandas.pydata.org/docs/index.html |
| requests   | 2.31.0  | https://pypi.org/project/requests/        |
| seaborn    | 0.12.2  | https://seaborn.pydata.org/               |

## Results
The amount of sex data is likely to be higher for clearly sexually dimorphic species. To test this, the script extracts species for which the total number of records is over 10,000 to provide a substantial sample upon which to calculate proportions. The proportion of male plus female records was compared with the total number of all records for both sexually monomorphic (23) and dimorphic species (31) in this subsample. For monomorphic species there is an average of about 5.5 records with a recorded sex per 1000 records  total, whereas for dimorphic species that average is 138.7; 25 times more.

Most dimorphic species have a higher proportion of males, in some cases well over two males for each female (Table 1). However, two species *Mergellus albellus* and *Somateria mollissima* have more females than males.

Table 1. The proportion of males of the most widespread sexually dimorphic ducks in Europe.

| Species                               | Vernacular name   | proportion of males |
|---|---|:-:|:-:|
| *Anas platyrhynchos* L., 1758     | Mallard           |                 1.54 |
| *Aythya fuligula* (L., 1758)      | Tufted Duck       |                1.39 |
| *Anas crecca* L., 1758            | Eurasian Teal     |               1.54 |
| *Mareca strepera* (L., 1758)      | Gadwall           |             2.29 |
| *Bucephala clangula* (L., 1758)   | Common Goldeneye  |                 1.12 |
| *Mareca penelope* (L., 1758)      | Eurasian Wigeon   |                 2.10 |
| *Spatula clypeata* (L., 1758)     | Northern Shoveler |                2.52 |
| *Somateria mollissima* (L., 1758) | Common Eider      |                 0.53 |
| *Aythya ferina* (L., 1758)        | Common Pochard    |                2.14 |
| *Mergus merganser* L., 1758       | Common Merganser  |                1.41 |

Since the turn of the millennium the volume of records with sex information has increased considerably (Fig. 1). The proportion of male birds is larger in recent decades, and generally there are more males than females, at least for the decades after the 1970s (Fig. 2). Exceptionally, in the 1950s there were considerably more females than males. This can be attributed to two datasets, both of which related to ringing and recovery data [@10.15468/6l4ban; @10.15468/yf8wjf].

![The total number of male and female records of for *Anas platyrhynchos* in Europe aggregate to decade](./figures/count.jpg) 

![A time series of duck sex ratio for *Anas platyrhynchos* in Europe aggregate to decade](./figures/ratiotimeseries.jpg) 

*Anas platyrhynchos* is widely distributed across Europe, however, records that hold sex information are not homogeneously distributed. For example, Denmark, Estonia and Norway have large quantities of sex data, while Ireland, Portugal, Spain and many Eastern European countries have little (Fig. 3). Most countries have a higher proportion of males than females, with the exception of Finland and France. In Finland the data largely come from a ringing and recovery dataset [@10.15468/kht1r3] containing both records of males and females. In France the data also largely come  from a single ringing and recovery dataset [@10.15468/yf8wjf]. However, in this case, although the dataset contains over two million records, none of them are annotated as male, and about half a million are annotated as female. The large patches of female dominated areas in France, such as in the Camargue, are the result of this dataset (Fig. 3), as is the anomaly in the 1950s mentioned earlier. In addition, a notable feature of the distribution of sex data is that clusters of data in large cities, including London, Madrid, Prague, Paris and Vienna.

![A map of the percentage of male ducks for *Anas platyrhynchos* in Europe aggregate to a 10 km using the EEA grid](./figures/duckmap.jpg)

Owing to the patchy distribution of records, interpolation can be a useful way to illustrate patterns in sex ratio, because it smooths some of the roughness of the data and gives more local weight to isolated points (Fig. 4). 

![An interpolated map of the proportion of males of *Anas platyrhynchos* and the associated kriging error](./figures/european_males_interpolation_and_error.jpg) 

## Discussion
Geospatial and temporal visualisation has unveiled insights into the sex ratio distribution of Anatidae across Europe. Visualisation of male and female counts over decades has revealed discernible patterns and anomalies, indicating both temporal fluctuations and spatial variations (Fig. 1 & Fig. 2).

Most species in our results are male biassed and that is consistent with other estimates in the literature [@Wildfowl731; @10.1007/s10336-019-01682-7; @10.1111/j.1474-919X.2007.00724.x]. In the case of *Somateria mollissima* where ratios tend to be female biassed, this is also consistent with the literature [@10.2981/0909-6396].

The distribution map shows that sex data are not evenly distributed across Europe (Fig. 3). There are national differences in the collection and mobilisation of data, which are likely the cause of this [@10.1016/j.biocon.2017.12.024]. The  concentration of records from large urban areas may be the result of records from citizen scientists who tend to focus more on developed areas than professional scientists [@10.1002/2688-8319.12185]. Furthermore, the apparent female bias of datasets from ringing is evident. This may be because females are more available for ringing and recapture when they are brooding, nursing their chicks, and while they are moulting. It would be possible to remake the data cube excluding ringing and recapture the dataset to avoid this bias.

However, a significant challenge in accurately assessing these ratios stems from the differential detectability of males and females in many species. Factors such as behavioural differences, sexual dimorphism, and varying habitat preferences can substantially influence the likelihood of observing and recording individuals of each sex. This variability in detectability complicates the interpretation of raw sex ratio data, potentially skewing our understanding of population dynamics. By aggregating vast amounts of occurrence data, these cubes potentially allow for the application of analytical techniques to address the issue of differential detectability. 

The use of biodiversity data cubes derived from GBIF data represents a novel approach to biodiversity studies, allowing for large-scale analysis that was previously more time consuming. This research not only contributes insights into sex ratio analysis but also demonstrates the potential of biodiversity data cubes in advancing ecological and conservation science.

## Conclusion
This study underscores the importance of biodiversity data cubes in facilitating large-scale analyses of ecological phenomena. By exploring sex ratio variations within European animal populations, this research contributes to the broader understanding of biodiversity patterns and their underlying mechanisms. The insights derived from this analysis advocate for the continued development and utilisation of biodiversity data cubes, highlighting their significance in ecological research and conservation strategies.

## Acknowledgements
We extend our gratitude to the Global Biodiversity Information Facility (GBIF) for providing the data that formed the basis of this research, and to the myriad contributors whose efforts in data collection and curation have enriched the GBIF repository.

## References
