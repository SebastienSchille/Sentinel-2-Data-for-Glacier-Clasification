<div style="display: flex; justify-content: center; gap: 20px;">
  <img src="/Images/ndsi_deployment_region.png" width="400" height="auto"/>
  <img src="/Images/Glacier_retreat_2.png" width="400" height="auto"/>
</div>

# Monitoring Glacier Retreat in the Swiss Alps Using Supervised and Unsupervised Machine Learning.

# 1.0 Project Background

Glaciers are extremely sensitive to climate change and can be studied to highlight the rapid change in alpine landscape within the 21st Century (Sommer et al., 2020). There has been an alarming increase in glacier retreats around the globe, leading to catastrophic consequences on mountain ecosystems, freshwater supplies, tourism, and local economies. Accurate monitoring is not only crucial for assessing the impacts of climate change but also for informing mitigation and adaptation strategies (Paul et al., 2007).

Traditional data collection methods, such as sample collection and manual interpretation of satellite data, can be highly time-consuming and often provide limited insight. Earth observation satellites launched within the Copernicus program have provided scientists with various datasets for a multitude of applications. Artificial intelligence can help refine models used to interpret results, improve understanding, and automate the process.

This project will focus on the Swiss Alps and the change in glacier ice coverage between 2017 and 2023 using Sentinel-2 data. The Normalised Difference Snow Index (NDSI) will be used as a benchmark and for training the CNN model. Unsupervised and supervised AI models will be deployed, with results quantified and discussed. Additionally, this project hopes to demonstrate proof of concept with a model that is transferable to other regions and discuss the limitations and recommended model changes.

## 1.1 Study Area & Data Collection

The data for this project has been sourced using the 'Data Collection.ipynb' available in the GitHub repository. The data was collected using the Copernicus database, with the region of interest covering the Swiss Alps, focusing on the Zermatt and Saas-Fe valleys.

The following dates were selected:

 * 15/08/2017
 * 20/07/2023 

 The following satellite data files for the dates above were downloaded:

 * S2A_MSIL2A_20170815T102021_N0500_R065_T32TMS_20231005T191904.SAFE
 * S2B_MSIL2A_20230720T101609_N0509_R065_T32TMS_20230720T131906.SAFE

---

<p align="center">
  <img src="/Images/area_of_interest.png" width="1000" height="auto"/>
</p>

**Figure 1:** The areas selceted for model training, validation and deploment. The darker areas in the SWIR band will represent snow ice in this case.

## 1.2 Aims & Objectives

**1.** To collect Sentinel-2 satellite imagery data from the region of interest in the summer months (2017 & 2023) to ensure cloud-free, comparable data for glacier analysis

**2.** To calculate the Normalised Difference Snow Index (NDSI) and Normalised Difference Snow and Ice Index (NDSII) as benchmark indicators of snow and glacier-covered areas

**3.** To apply k-means algorithm as an unsupervised method for initial classification of areas with glacier ice cover.

**4.** To train a Convolutional Neural Network (CNN) using labelled data derived from a mask created using the benchmark indices.

**5.** To validate the machine learning outputs by comparing them to NDSI/NDSII-based classifications.

**6.** To quantify changes in glacier cover between 2017 & 2023 by analysing the differences between glacier masks.

**7.** To evaluate the accuracy and limitations of unsupervised and supervised learning for glacier monitoring, including the scalability and potential model improvements.
   
# 2.0 Methodology

The figure below illustrates the workflow of the project:

<p align="center">
  <img src="/Images/Flowchart for project.png" width="800" height="auto"/>
</p>

**Figure 2:** Illustration of the methodology and implementation of AI in this project.

## 2.1 Sentinel-2 Optical sensors

Sentinel-2 is equipped with a with an advanced Multispectral Instrument (MSI). This sensor captures high-resolution imagery across 13 spectral bands, specifically designed for earth observation applications. This project will use the bands from the table below:

| Band | Name                | Wavelength (nm) | Resolution | Use for Glacier Mapping                           |
|------|---------------------|------------------|-------------|---------------------------------------------------|
| B2   | Blue                | 490              | 20 m        | Atmospheric correction, snow detection            |
| B3   | Green               | 560              | 20 m        | Used in **NDSI** to detect snow/ice               |
| B4   | Red                 | 665              | 20 m        | General land cover classification                 |
| B8A  | Narrow NIR          | 865              | 20 m        | Used in **NDSII**, better glacier detection       |
| B11  | SWIR                | 1610             | 20 m        | Used in **NDSI & NDSII**, ice and snow absorption |

**Table 1:** Sentinel-2 optical bands used in this project (SentiWiki, 2025).

## 2.2 Snow and Ice Indices

This project uses a combination of indices that have been developed for snow and ice detection. The Normalised Difference Snow Index (NDSI) is used to identify snow-covered areas by using the green band and the SWIR reflectance (SentiWiki, 2025). Snow and ice reflect green visible light while they are highly absorbative of short-wave infrared (SWIR). Additionally, clouds reflect the SWIR band enabling to disregard them as snow or ice. The NDSI can be calculated from the equation below:

**NDSI** = (B3 − B11) / (B3 + B11)  

*Where:*  
- **B3** = Green band (~560 nm)  
- **B11** = Short-Wave Infrared (SWIR) band (~1610 nm)

<p align="center">
  <img src="/Images/NDSI_calculation.png" width="1000" height="auto"/>
</p>

**Figure 3:** NDSI calculated for the three areas of interest.

However, this index will struggle to distinguish between snow and ice, leading to misidentification. Additionally, in Figure 3 the NDSI index has labelled a water body as glacier ice in the deployment region. To combat this issue the Normalised Difference Snow and Ice Index (NDSII) can be used (Keshri, 2008). This index helps to differentiate between snow and ice by using the NIR band. Glacier ice will reflect the NIR band more than snow hence aiding to distinguish them.

**NDSII** = (B11 − B8A) / (B11 + B8A)  

*Where:*  
- **B11** = Short-Wave Infrared (SWIR) band (~1610 nm)  
- **B8A** = Narrow Near-Infrared (NIR) band (~865 nm)

<p align="center">
  <img src="/Images/NDSII_calculation.png" width="1000" height="auto"/>
</p>

**Figure 4:** NDSII calculated for the three areas of interest.

---

The combination of these two indices was used to create a benchmark mask.
The threshold is set as: glacier_mask = (ndsi > 0.4) & (ndsii > 0.3). In turn the use of the NDSII has corrected the misidentification of the water body.

<p align="center">
  <img src="/Images/NDSI_NDSII_mask.png" width="1000" height="auto"/>
</p>

**Figure 5:** Masks created and labelled for glacier ice using the NDSI & NDSII indices.

## 2.3 K-means learning

K-means classification is an unsupervised machine learning tool that doesn't require labelled data. It will compare neighbouring pixels to find trends, patterns and similarity. The initialisation, location, and parameters of the centroids used in the model can be changed to meet the user's needs (IBM, 2024). The model can be adjusted to change the number of clusters or groups to be identified. In this project, this method was used as a baseline classifier to find ice-covered areas.

<p align="center">
  <img src="/Images/K_means_figure.png" width="600" height="auto"/>
</p>

**Figure 6:** Illustration of K-means machine learning algorithm (Jeffares, 2019).

---
K-means was used to classify glacier ice and to generate the mask show in Figure 7. The confusion matrix (Figure 8) shows a high accuracy with the mask created by the benchmark indices, however, there are some clear visual differences. The algorithm has successfully disregarded the water body in the deployment area however due to limited model depth it will struggle in more complex region especially determining between snow and ice. Therefore, it is preferable to train a CNN model that can be developed for higher accuracy.

<p align="center">
  <img src="/Images/K_means_mask.png" width="1000" height="auto"/>
</p>

**Figure 7:** Masks created using K-means for Ice detection.

<p align="center">
  <img src="/Images/K_means_confusion_matrix.png" width="500" height="auto"/>
</p>

**Figure 8** Confusion matrix to compare with the NDSI & NDSII benchmark mask.


## 2.4 CNN model

Convolutional Neural Networks (CNNs) are a type of supervised machine learning model that are trained using labelled datasets. These networks are particularly well-suited for classification tasks, such as image recognition, object detection, and segmentation, because they are specifically designed to process data with a grid-like topology. CNNs operate by applying convolutional filters across the input data, allowing them to automatically detect and learn important patterns, trends, textures, and spatial hierarchies within the images (IBM, 2025). This capability enables CNNs to extract features from simple edges to complex shapes. In this project, a CNN model will be trained using labelled data derived from benchmark indices alongside Sentinal-2 optical bands listed in Table 1.

<p align="center">
  <img src="/Images/CNN_figure.png" width="600" height="auto"/>
</p>

**Figure 9** Illustration of the convolution layers in CNN models.

# 3.0 Results & Discussion

The snow and ice indices used for this project were used to create benchmark masks for the machine-learning models. The selection of adequate thresholds is often unclear and must be assessed for each area of interest - limiting scalability and increasing manual labour. To automate the process, AI models can be used to learn from these indices and be trained on varying regions with different landscapes and conflicting objects, such as bodies of water and clouds. In this project, the Normalised Difference Snow Index (NDSI) was set with a threshold of >0.4 which removed fresh snow along with other misidentified features, while including the older glacier ice. When the threshold was increased to >0.6 older debris-covered glacier ice was not considered as ice due to its lower reflectance in the green band. However, this lower threshold included a body of water in the lower right of the deployment region. To attempt to remove this feature an index from (Keshri, 2008) stated as the Normalised Difference Snow and Ice Index (NDSII) was applied in conjunction. The use of the additional NIR band allowed for the removal of the water body due to its low reflectance. In conclusion, indices used to categorise land areas and identify features are a powerful tool and require little processing power, however, the correct application is essential to ensure accurate results.

The masks created by the indices (shown in Figure 5) were used to train the CNN model on the test region. This model was then rolled out on the validation region with the results shown in Figure 10. The CNN model was able to identify glacier ice with high accuracy when compared to the index benchmark.

<p align="center">
  <img src="/Images/CNN_validation_region.png" width="1000" height="auto"/>
</p>

**Figure 10** CNN model rollout on the validation region.

However, when the CNN model was rolled out on the deployment region the water body was misidentified as glacier ice. This is likely due to the training area not including a water body for initial training. This highlights the importance of a large and varied training dataset for a CNN model. This requires manual labour time and large processing cost.

<p align="center">
  <img src="/Images/CNN_deployment_region.png" width="1000" height="auto"/>
</p>

**Figure 11** CNN model rollout on the deployment region.

The masks created from the rollout of the CNN model on the deployment region in 2017 and 2023 were used to quantify glacier retreat. Each pixel was compared to understand if glacier ice had been lost or gained with the difference quantified to find an overall loss of 26.59% - shown in Figure 12.

<p align="center">
  <img src="/Images/Glacier_retreat.png" width="600" height="auto"/>
</p>

**Figure 12** Glacier retreat (difference between 2017-2023 summer).

Ice loss in the region can be predicted by using the exponential decay function with the assumption of a 26.59% of ice loss in 6 years, leading to an exponential decay of 0.0515. This method is able to highlight the alarming glacier retreat rate, however, due to a multitude of complex variables it isn’t an accurate prediction.

<p align="center">
  <img src="/Images/Projected_ice_loss.png" width="600" height="auto"/>
</p>

**Figure 13** Projected glacier loss using an exponential decay model.

It is important to reiterate the limitations of this model and its sensitivity to debris, cloud cover and bodies of water. The model in this project is a proof of concept for the application and scalability in the industry. To improve the model, a larger training data set should be used on varying regions and conditions. Pre-labelled data from scientific organisations could also help train and validate the models. The CNN model can also be modified to use a larger convolutional kernel size to include spatial data and the extraction of more information from the layers.

# 4.0 Project enviromental Cost

For simplification I have divided the environmental cost of this project into the following sections:
1. **AI model training**
2. **Computing power**
3. **Cloud services**
4. **Indirect emissions**

## 4.1 AI model training

AI models require large computational power to train and validate the models built. To ensure the accuracy of these models, large data sets must be used along with many epochs. The estimation for the consumption of these models can be done with several programs created such as carbontracker (Anthony et al., 2020). The code created enables integration into the user’s code and will output total consumption. The output shown in Figure 14 is the estimated emissions related to training my AI model. However low the emission seem, the model was re-trained many times and was a simplified proof of concept model compared to what would need to be built for industry applications.

<p align="center">
  <img src="/Images/CNN_emissions.png" width="400" height="auto"/>
</p>

**Figure 14** AI model training emissions estimated by ‘carbontracker’ (Anthony et al., 2020).

## Computing power

Computer power will be the main source of emissions for this project, with access to the internet, research, and coding required. In the UK 1Kwh is responsible for 124g CO2 emissions, which is down by 74% since 2014 (Evans & Tovey, 2025). Under the assumption that a computer will use around 100W of energy, this will lead to approximately 496g CO2 emissions weekly (8hr workday day, 5 days a week).

### CO₂ Emissions Calculation

**Given:**
- Weekly energy use: 4 kWh
- UK carbon intensity (2024): 124 gCO₂/kWh

**Calculation:**
CO₂ emissions = 4 kWh/week × 124 gCO₂/kWh = 496 gCO₂/week

Which is equivalent to: **0.496 kg CO₂/week**

## Cloud services

Emissions from cloud services will come from data centres, data transmission and user devices. Cloud services run on large data centres with servers that work 24/7 and require huge amounts of energy to power and cool the equipment. When data is transmitted by uploading or downloading files, this requires a large network infrastructure, including cell towers and routers. Google has been committed to transitioning the power to its data centres to renewable sources (Google, 2024). The environmental impact of data services is not well understood, with misleading articles and figures easily found on the internet. However, the International Energy Agency has estimated that they account for 1-1.5% of global emissions and that the transition to renewable energy will be essential to curb the growth of energy demand over the next decade (International Energy Agency, 2023).

## Indirect emissions

For the completion of the project, additional indirect emissions should be considered:

-	Energy used in manufacturing and transport of equipment used
-	Transport and building energy use for the work environment
-	Supply chain emissions from third-party service providers
-	Waste disposal related to project materials

# Project video

Link to video: https://youtu.be/9avymYPL48U

# Acknowledgments

This project was created for the final project part of the AI for Earth Observation (GEOL0069) module taught by Dr Michel Tsamados at UCL.

# References
Anthony, L. F. W., Kanding, B., & Selvan, R. (2020). Carbontracker: Tracking and predicting the carbon footprint of training deep learning models. ICML Workshop on Challenges in Deploying and Monitoring Machine Learning Systems. https://github.com/lfwa/carbontracker

European Space Agency. (2025). Sentinel-2 operations. ESA. https://www.esa.int/Enabling_Support/Operations/Sentinel-2_operations

Evans, S., & Tovey, M. (2025). Analysis: UK’s electricity was cleanest ever in 2024. Carbon Brief. https://www.carbonbrief.org/analysis-uks-electricity-was-cleanest-ever-in-2024/

Google. (2024). Google’s 2024 environmental report. https://sustainability.google/reports/google-2024-environmental-report/

IBM. (2024). What is k-means clustering? https://www.ibm.com/think/topics/k-means-clustering

IBM. (2025). What are convolutional neural networks? https://www.ibm.com/think/topics/convolutional-neural-networks

International Energy Agency. (2023). Data centres and data transmission networks. https://www.iea.org/energy-system/buildings/data-centres-and-data-transmission-networks

Jeffares, A. (2019). K-means: A complete introduction. Towards Data Science. https://medium.com/data-science/k-means-a-complete-introduction-1702af9cd8c

Keshri, A. K., Shukla, A., & Gupta, R. P. (2008). ASTER ratio indices for supraglacial terrain mapping. International Journal of Remote Sensing, 30(2), 519–524. https://doi.org/10.1080/01431160802385459

Paul, F., Kääb, A., & Haeberli, W. (2007). Recent glacier changes in the Alps observed by satellite: Consequences for future monitoring strategies. Global and Planetary Change, 56(1–2), 111–122. https://doi.org/10.1016/j.gloplacha.2006.07.007

SentiWiki. (2025). S2 applications. Copernicus. https://sentiwiki.copernicus.eu/web/s2-applications

Sommer, C., Malz, P., Seehaus, T. C., et al. (2020). Rapid glacier retreat and downwasting throughout the European Alps in the early 21st century. Nature Communications, 11, 3209. https://doi.org/10.1038/s41467-020-16818-0


