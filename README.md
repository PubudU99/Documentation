## UMT changes for the CST process

### Changes done in UMT UI

- Add seperate columns for the CST build status and the CST build Action.

![UI Image](docs/images/ui/ui_img01)

Changed the backend to retrieve the CST build status for the specifc chunks that is not yet released.


### Changes done in UMT Backend
1. Add new configurations for the CST ballerina service.
2. Implement two POST requests for triggering the CST CI-CD pipeline and get cst pipeline status of a specific chunks.
3. Create two tables to save pipeline status and history table for the cst_status table
     - ```release_chunk_cst_build_status``` table for saving pipeline status.
     - ```release_chunk_cst_build_status_history``` table for track the update in ```release_chunk_cst_build_status``` table.

```release_chunk_cst_build_status```

![table01](docs/images/umt-backend/be_img01)

```release_chunk_cst_build_status_history```

![table02](docs/images/umt-backend/be_img02)

**How the triggering tests process works**

Step 01

UMT - UI calls for triggering the tests including Integration tests, Test grid and the CST process.

In CST process..

Step 02

Backend logs that request is hit by the backend.

![backendlogs01](docs/images/umt-backend/be_logs_img01)

Step 03
Then the update chunk details are being converted as the how the ballerina backend requires.

e.g :- 

This is how the release chunk table stored the release chunk details.

```sync_response```
```
[
  {
    "product-name": "wso2am",
    "product-version": "3.2.0",
    "channel": "full",
    "update-level": 120,
    "applied-updates": [615]
  },
  {
    "product-name": "wso2mi",
    "product-version": "1.0.0",
    "channel": "full",
    "update-level": 6,
    "applied-updates": [615]
  }
]
```
Those details should be converted as below before send to the ballerina service.

```productNameandVersion```

```
[
  {
    "productBaseVersion": "3.2.0",
    "productName": "wso2am"
  },
  {
    "productBaseVersion": "1.0.0",
    "productName": "wso2mi"
  }
]
```

Then UMT backend checks the response of the ballerina service and save the result in the ```release_chunk_cst_build_status``` table.

**How the status update process works**```release_chunk_cst_build_status_history```

This is implemented to get the CST status of the specific chunks.

Step 01

First this checks for which releases that the status needs to be update.
This logic checks for which updates are still not released yet but created.

After getting the ```chunk id```s of the updates which are still on going, then get the specific ```UUID``` for that specific chunk from the  ```release_chunk_cst_build_status``` table.

Step 02

After getting the ```UUID``` of the chunk/s that the status need to be updated, ```UUID``` is attached to the request and send to the ballerina backend. Then it gives the status for each pipeline in the release chunk.

Then UMT backend checks the response of the ballerina service and saves the status in the ```release_chunk_cst_build_status``` table and updates the ```release_chunk_cst_build_status_history``` table.





