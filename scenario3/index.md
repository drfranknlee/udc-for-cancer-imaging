# Scenario 3: Extend Catalog to Data & AI Workbench



The step-by-step instruction and results are captured with screenshots and can be downloaded as [PDF](recording/T101389-Scenario3-v20210921.pdf) or [PPT](recording/T101389-Scenario3-v20210921.pptx)



## 1. Browse Dataset for Export

As images  coupled in a pair of "raw" vs "annotated" are now all tagged with the same u2-id value, we will just search for a pair of images using the Visual Query as shown. 

<img src=recording/T101389-Scenario3-visualsearchforexportdata.png>


Here is the result of the search

<img src=recording/T101389-Scenario3-visualsearchforexportdataresult.png>


Now we can use the Export-to-WKC function to select the pair of images along with tags to export to a chosen catalog in Cloud Pak for Data. The connection between Spectrum Discover and WKC should be set according to the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover) 


<img src=recording/T101389-Scenario3-exporttowkc.png>


## 2. Work with Exported Dataset in Cloud Pak for Data


### 2.1 Work with Data in WKC

The exported metadata, data connection (to the underlying Cloud Object Storage where the data is physically stored), required credential to access the data, are now exported and available inside Watson Knowledge Catalog (WKC). WKC is the enterprise data catalog and platform inside Cloud Pak for Data (CP4D). 

<img src=recording/T101389-Scenario3-cp4d.png>


You should be able to see the data inside WKC Catalog. Select the "Recently-added" tab in the navigation bar to find the data asset most-recently imported. 

<img src=recording/T101389-Scenario3-datasetinwkc.png>


Then you can work with invidual data asset by visiting its Overview, Asset, Review, Activities and also setting up access control and sharing for the data asset. In the Overview page, you can also see the metadata tags exported from Spectrum Discover and the path to the cloud object storage. 

<img src=recording/T101389-Scenario3-wkc-file-overview.png>

In the Asset page, you can preview certain types of files such as images (.jpg, .png, etc). In our case, both images (annotated with a bounding box of the aircraft, or the raw image) can be viewed directly. 

<img src=recording/T101389-Scenario3-wkc-file-access-view.png>


### 2.2 Work with Data in Watson Studio

In this example, you can first export the master manifest file (T101389_s2_manifest.csv) from Spectrum Discover to WKC. Then you can publish the file to Watson Studio Project as a data asset. 

<img src=recording/T101389-Scenario3-publish-data-from-wkc-to-ws-project.png>


Once landed in Watson Studio, there will be many options to analyze and visualize the data asset. Here you can use Data Refinery function to refine the manifest. 

<img src=recording/T101389-Scenario3-data-refinery-refine.png>


Then the refined manifest can be visualized using many of the built-in graphical visualization engines

<img src=recording/T101389-Scenario3-data-refinery-visualize.png>