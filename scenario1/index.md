# Scenario 1: Create Tags from AutoTag Policy, Custom-Tagging and Filtering


The step-by-step instruction and results are captured with screenshots and can be downloaded as [PDF](rm/T101389-s1-v20210928.pdf) or [PPT](rm/T101389-s1-v20210928.pptx)



## 1. Set up Spectrum Discover to Manage Data Sources
Please consult the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover) on setting up and customizing Spectrum Discover and connecting/scanning a target data source. When all set up, you can manage the access to external data sources (S3, File Systems), initiate and review the scanning of target resources. 

<img src=rm/T101389-s1-sd-data-source-management.png>


## 2. Upload Dataset to Object Storage

In your local or public cloud account, create a cloud object storage service. Start with a new vault called udc-vault and create a folder within called "T101389". All your demo data will be uploaded into this folder. This object storage should be set up as a managed data source by Spectrum Discover. 

<img src=rm/T101389-s1-upload-data.png>


## 3. Set up AutoTag Policy in Spectrum Discover

In this step, you will set up a Spectrum Discover AutoTag policy to extract the image source (eg. raw vs annotated) from the path of the images and polulate the value into metadata tag named "u2-source".


SSH log into the Spectrum Discover server. If you cannot gain SSH access to the server, you can still create the policy by using the Spectrum Discover RESTful API after obtaining the token. Please consult the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover)


Create the policy json file as following

        {
        "pol_id": "T101389_autotag_u2source_pol",
        "action_id": "AUTOTAG",
        "action_params": {
            "rule": {
            "name":"setFromExistingField",
            "action": "regex",
            "existingField": "filename",
            "regexPattern": "[^/]+",
            "matchNo": 3,
            "newField": "u2-source"
            }
        },
        "schedule": "NOW",
        "pol_filter": "filename LIKE ('%T101389%') AND filetype LIKE ('%jpg%')",
        "pol_state": "active",
        "explicit": "true",
        "collections": []
        }



Run first command to refresh the token.

   
    gettoken


Run the second command to create the policy using RESTful API service.

    
    curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyengine/v1/policies/T101389_autotag_u2source_pol -X POST -d @./T101389_autotag_u2source.json -H "Content-Type: application/json"
    


This will create a Spectrum Discover AutoTag policy named "T101389_autotag_u2source_pol". The policy will also be executed automatically. 

Use the query builder to search for the files: 

    filename LIKE ('%T101389%') AND filetype LIKE ('%jpg%') 


The matched files are displayed with the tag "u2-source" showing values of "annotated" or "raw". The values were extracted by AutoTag policy from the 2nd level of filename or fileppath. 

<img src=rm/T101389-s1-autotagdataset.png>


## 4. Custom-tag New Dataset

To preserve the collection of files processed by AutoTag policy above as a dataset, you'll use custom-tagging to populate "u2-dataset" tag with value "T101389-s1". By doing so, you can find this collection of data quickly. 

Click on the "select all" box to pick all the files and then click on "add tags". 

<img src=rm/T101389-s1-04-select-files-to-add-tags.png>


Use the dialogue box to select the tag (eg "u2-dataset") and enter the value "T101389-s1":

<img src=rm/T101389-s1-04-add-cutom-tag.png>

The files and the newly-tagged field "u2-dataset" is displayed after refreshing the web browser: 

<img src=rm/T101389-s1-customtagnewdataset.png>


You can export the list of files tagged by "u2-dataset=T101389-s1" as a report in [CSV file](rm/T101389-s1-manifest-v20210928.csv)

<img src=rm/T101389-s1-exporttomanifest.png>

<img src=rm/T101389-s1-exporttomanifest2.png>

The report is stored and can be downloaded from Spectrum Discover web console. You will use this manifest in other scenarios as a file manifest.

<img src=rm/T101389-s1-04-manifest-report-stored-in-discover.png>


## 5. Create New Tags Based on Filters

In this step, you will use AutoTage policy to create two new tags based on filtering. The filtering is a set of conditions you can specify to effectively calculate the qualifying files to be tagged. 

Below is the AutoTag policy:

    {
        "pol_id": "E1016404_autotag_add_field_pol",
        "action_id": "AUTOTAG",
        "action_params": {		
        "tags": {"u2-el":"E1016404", "u2-pp":"FL"}
 
        },
        "schedule": "NOW",
        "pol_filter": "filename LIKE ('%T101389%') AND path LIKE ('%udc-vault%') AND u2-source LIKE ('%annotated%')",
        "pol_state": "active",
        "explicit": "true",
        "collections": []
    }


In Spectrum Discover server, run first command to refresh token:

    gettoken


Run the second command to create the AutoTag policy:

    curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyenginolicies/E1016404_autotag_add_field_pol -X POST -d @./E1016404_autotag_add_field_pol.json -H "Content-Type: application/json"


In the Spectrum Discover console, you can confirm the policy has now been created and executed automatically for the first time.

<img src=rm/E1016404-10-autotag-policy-added.png>

In the Policy History tab, you can confirm that the policy has run successfully. 

<img src=rm/E1016404-20-autotag-policy-run-completed.png>


You can now use the same filtre in the policy to find the files. 

<img src=rm/E1016404-30-query-for-new-dataset.png>

The matched files are displayed along with the two new tags -- <B>u2-el</B> and <B>u2-pp</B>

<img src=rm/E1016404-40-new-dataset-w-calculated-tag.png>



## 6. View Images Directly with Calculated URL

Combine the values of metadata tag "path" (ie bucket/vault) and "name" (filepath to the objects including the directories) for the full path to the images stored in the object storage bucket. The url for the object storage server or host can be located in the Spectrum Discover "Data source management" panel. In our case:

- path: udc-vault
- name: T101389/dataset/annotated/000ec980b5b17156a55093b4bd6004ab.jpg
- cluster: 9.11.221.105

<img src=rm/T101389-s1-find-fullpath-to-object.png>


You can concatenate the cluster, path and name value to the url for the object as: http://9.11.221.105/udc-vault/T101389/dataset/annotated/000ec980b5b17156a55093b4bd6004ab.jpg


You can open and visualize the image direclty from a web browser. 

<img src=rm/T101389-s1-direct-access-to-file.png>

