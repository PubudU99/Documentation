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

How the process works
Step 01
UMT - UI calls for triggering the tests including Integration tests, Test grid and the CST process.

In CST process..
Step 02
Backend logs that request is hit by the backend.

![backendlogs01](docs/images/umt-backend/be_logs_img01)

Step 03
Then the update chunk details are being converted as the how the ballerina backend requires.
e.g :- 




