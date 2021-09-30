# Scenario 2: Import Metadata to Enrich Catalog



The step-by-step instruction and results are captured with screenshots and can be downloaded as [PDF](recording/T101389-Scenario2-v20210921.pdf) or [PPT](recording/T101389-Scenario2-v20210921.pptx)



## 1. Upload New Dataset Manifest

Upload the new dataset "T101389-s2" manifest from local /dataset directory to "/udc-vault/T101389/dataset/"

> Note: You can use your own dataset as long as the path to each file/object in the object storage bucket is updated



## 2. Set up Tag-import Policy in Spectrum Discover

SSH log into the Spectrum Discover server. If you cannot gain SSH access to the server, you can still create the policy by using the Spectrum Discover RESTful API after obtaining the token. Please consult the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover)


Create the policy json file as following

        {
                "pol_id": "T101389_ImportTags_pol",
                "action_id": "IMPORT_TAGS",
                "action_params": {
                        "agent":"ImportTags",
                        "source_connection":"UDSt-CoS",
                        "tag_file_path":"udc-vault/T101389/dataset/T101389_s2_manifest.csv",
                        "tag_file_type":"csv"
                },
                "schedule": "NOW",
                "pol_state": "active",
                "pol_filter": "datasource IN ('udc-vault')"
        }


Run two commands from CLI:

        gettoken

        curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyengine/v1/policies/T101389_ImportTags_pol -X POST -d @./T101389_ImportTags_pol.json -H "Content-Type: application/json"


This will create a Spectrum Discover metadata policy "IMPORT_TAGS" named "T101389_aircraft_import_scenario2_pol". The policy will also be executed automatically. 




## 3. Use the Enriched Data Catalog


### 3.1 Find or browse the newly-imported metadata 

Use the query builder to search for the dataset: 

    filename LIKE ('%T101389%') AND filetype LIKE ('jpg')


The serach result is shown below. Note that we can now show additional matadata tags such as the dimensions of the bounding box (xmin, xmax, ymin, ymax) and image (width, length) for the aircraft, the type of aircraft (u2-type), the image ID (u2-id) that we assigned to each image in the manifest. 

<img src=recording/T101389-Scenario2-importtagresult.png>


### 3.2 Custom-tag the new dataset

Once the dataset is found and displayed, select those that meet the criteria of a new dataset to be custom-tagged. In our case shown in the recording, we will select all 22 images for tagging. We'll use tag "udc-dem3" and value "T101389-s2" to create the the new dataset. 

<img src=recording/T101389-Scenario2-customtag.png>


### 3.3 Use Visual Query to browse new dataset

With these newly-available tags such as u2-type, u2-id etc, we can now go to Spectrum Discover Visual Query search panel to browse for the type and quantities of data in each group/metrics. 


<img src=recording/T101389-Scenario2-visualsearchtype.png>


### 3.4 Custom-tag the new dataset

In Spectrum Discover, we can now also pull up a pair of images (raw and annotated) based on their image ID tag "ud-id"

    u2-id LIKE ('%T101389-10005%')


See the resulting display of the pair of data in the catalog, and the images superimposed next to each other with the bounding box for the annotated one. 

<img src=recording/T101389-Scenario2-finding-pair-1.png>

<img src=recording/T101389-Scenario2-finding-pair-2.png>
