# Scenario 1: Initialize Catalog with Metadata Ingest & Indexing, AutoTag and Custom-Tagging



The step-by-step instruction and results are captured with screenshots and can be downloaded as [PDF](rm/T101434-s1.pdf).



## 1. Set up Spectrum Discover to Manage Data Sources
Please consult the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover) on setting up and customizing Spectrum Discover and connecting/scanning a target data source. When all set up, you can manage the access to external data sources (S3, File Systems), initiate and review the scanning of target resources. 



## 2. Metadata Ingest and Default Indexing

In your local or public cloud account, create a cloud object storage service. Start with a new vault called udc-vault and create a folder within called "T101389". All your demo data will be uploaded into this folder. This object storage should be set up as a managed data source by Spectrum Discover. 

<img src=rm/01-upload-data-to-CoS.png>



Search for the newly-uploaded data in Spectrum Discover using SQL Query tool:

<img src=rm/02-discover-query-newly-added-and-tagged-dataset.png>


The data are indexed with system metadata auto-extracted by Spectrum Discover: 

<img src=rm/05-auto-indexed-ingested-dataset.png>



## 3. Set up AutoTag Policy in Spectrum Discover

In this step, you will set up a Spectrum Discover AutoTag policy to extract the image source (eg. raw vs annotated) from the path of the images and polulate the value into metadata tag named "u2-source".


SSH log into the Spectrum Discover server. If you cannot gain SSH access to the server, you can still create the policy by using the Spectrum Discover RESTful API after obtaining the token. Please consult the [Spectrum Discover Documentation](https://www.ibm.com/docs/en/spectrum-discover)


Create the AutoTag regex policy using json as following

        {
        "pol_id": "T101434_autotag_u2type_pol",
        "action_id": "AUTOTAG",
        "action_params": {
            "rule": {
            "name":"setFromExistingField",
            "action": "regex",
            "existingField": "filename",
            "regexPattern": "\/SOB_(.*)_d_",
            "matchNo": 1,
            "newField": "u2-type"
            }
        },
        "schedule": "NOW",
        "pol_filter": "filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')",
        "pol_state": "active",
        "explicit": "true",
        "collections": []
        }


The policy will use regex pattern to extract from the filename "T101434/dat/T100140/SOB_benign_d_adenosis_p_14-22549AB/100X/SOB_B_A-14-22549AB-100-001.png" the value of "benign". 


Run first command to refresh the token.

   
    gettoken


Run the second command to create the policy using RESTful API service.

    
    curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyengine/v1/policies/T101434_autotag_u2type_pol -X POST -d @./T101434_autotag_u2type.json -H "Content-Type: application/json"
    
<img src=rm/10-create-autotag-policy-default.png>

This will create a Spectrum Discover AutoTag policy named "T101389_autotag_u2source_pol". 

<img src=rm/15-autotag-default-policy-created.png>


The policy will also be executed automatically. 

<img src=rm/20-autotag-default-policy-executed-now.png>


You can view the tagged data using SQL Query tool:


    filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')


<img src=rm/21-u2type-tagged-by-autotag-policy.png>



You can also customize the policy to run with a specific schedule (eg. monthly, weekly, daily, hourly etc). Add a "schedule" block to the policy: 

        {
        "pol_id": "T101434_autotag_u2pp_pol",
        "action_id": "AUTOTAG",
        "action_params": {
            "rule": {
            "name":"setFromExistingField",
            "action": "regex",
            "existingField": "filename",
            "regexPattern": "_p_(.*)\/[124]0*X",
            "matchNo": 1,
            "newField": "u2-pp"
            }
        },

	"schedule": {
           "dayOfWeek": "*",
            "hour": "*",
            "minute": 5
        },

        "pol_filter": "filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')",
        "pol_state": "active",
        "explicit": "true",
        "collections": []
        }



Then create the policy with built-in schedule: 

	curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyengine/v1/policies/T101434_autotag_u2pp_pol -X POST -d @./T101434_autotag_u2pp.json -H "Content-Type: application/json"
    

<img src=rm/23-create-autotage-policy-with-schedule.png>


The policy is created and you can view its schedule in the Policy console. 

<img src=rm/25-autotag-auto-policy-created.png>


The policy will run automatically based on the schedule. 

<img src=rm/25-autotag-auto-policy-executed.png>


Use the query builder to search for the files: 

    filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')


The matched files are displayed with the tag "u2-pp" showing the value of patient number extracted with regex pattern "_p_(.*)\/[124]0*X".

<img src=rm/26-autotag-auto-policy-executed-result.png>


## 4. Custom-tag Dataset

To preserve the collection of files processed by AutoTag policy above as a dataset, you'll use custom-tagging to populate "u2-dataset" tag with value "T101434-s1". By doing so, you can find this collection of data quickly. 

Click on the "select all" box to pick all the files and then click on "add tags". 

<img src=rm/40-select-files-to-add-tags.png>


Use the dialogue box to select the tag (eg "u2-dataset") and enter the value "T101434-s1":

<img src=rm/45-add-tag-to-dataset.png>

The files and the newly-tagged field "u2-dataset" is displayed after refreshing the web browser: 

<img src=rm/46-tagged-dataset.png>


You can export the list of files tagged by "u2-dataset=T101434-s1" as a report in [CSV file](rm/T101434-s1-v20211010.csv)

<img src=rm/47-create-report-as-manifest.png>

<img src=rm/48-create-report-as-manifest.png>

The report is stored and can be downloaded from Spectrum Discover web console. You will use this manifest in other scenarios as a file manifest.

<img src=rm/49-report-created.png>



## 5. Automate Custom-tagging with AutoTag Policy

In this step, you will create AutoTage policy to automate custom-tagging based on filtering. The filtering is a set of conditions you can specify to identify a set of data for custom-tagging. With the policy, the tagging will be accomplished by populating values into selected metadata tags. 

Below is the AutoTag policy:

    {
        "pol_id": "T101434_autotag_add2_pol",
        "action_id": "AUTOTAG",
        "action_params": {		
        "tags": {"u2-el":"T101434-s1", "udc-dem2":"T101434-s1"}
 
        },
        "schedule": "NOW",
        "pol_filter": "filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')",
        "pol_state": "active",
        "explicit": "true",
        "collections": []
    }


In Spectrum Discover server, run first command to refresh token:

    gettoken


Run the second command to create the AutoTag policy:


	curl -k -H "Authorization: Bearer ${TOKEN}" https://localhost/policyengine/v1/policies/T101434_autotag_add2_pol -X POST -d @./T101434_autotag_add2.json -H "Content-Type: application/json"

    

In the Spectrum Discover console, you can confirm the policy has now been created and executed automatically for the first time.

<img src=rm/50-autotag-add2-policy-created.png>

In the Policy History tab, you can confirm that the policy has run successfully. 

<img src=rm/51-autotag-add2-policy-executed.png>


You can now use the same SQL query as defined in the policy filter to find the data. 

    filename LIKE ('%T101434%') AND datasource LIKE ('%udc-vault%')  AND filetype LIKE ('%png%')



The matched files are displayed along with the two new tags -- <B>udc-dem2</B> and <B>u2-el</B>

<img src=rm/53-autotag-add2-tagged-dataset.png>


## 6. View Images Directly with Calculated URL

Combine the values of metadata tag "path" (ie bucket/vault) and "name" (filepath to the objects including the directories) for the full path to the images stored in the object storage bucket. The url for the object storage server or host can be located in the Spectrum Discover "Data source management" panel. In our case:

- path: udc-vault
- name: T101434/dat/T100140/SOB_benign_d_adenosis_p_14-22549AB/100X/SOB_B_A-14-22549AB-100-001.png
- cluster: 9.11.221.105

<img src=rm/60-rebuild-object-access-url.png>


You can concatenate the cluster, path and name value to the url for the object as: http://9.11.221.105/udc-vault/T101434/dat/T100140/SOB_benign_d_adenosis_p_14-22549AB/100X/SOB_B_A-14-22549AB-100-001.png


You can open and visualize the image direclty from a web browser. 

<img src=rm/61-image-displayed-directly-from-url.png>

