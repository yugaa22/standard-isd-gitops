Below are the steps to be followed to upgrade to 3.11


1. User need to have ISD 3.10 version installed in the system.

2. You should have ISD 3.10 values.yaml in handy and need to update accordingly with 3.11 values and push to git repository.

**NOTE: User need to have the ISD n+1 version of values.yaml in the git repo. 

Example: If user is having 3.10 ISD version he should update 3.11 values accordingly and push to git reposirtoy. 

3. User first need to take the back up of the Databases (i.e Minio,Postgres,Redis).

  In the gitops-repo cloned to disk. cd to the gitops-repo cloned

  Use the sed commands to replace the pvc names in minio,postgres and redis folders

  sed -i 's/PVCNAME/<USER_SPECIFIED_VALUE>/g' redis/*.yaml

  sed -i 's/PVCNAME/<USER_SPECIFIED_VALUE>/g' minio/*.yaml

  sed -i 's/PVCNAME/<USER_SPECIFIED_VALUE>/g' postgres/*.yaml

**NOTE: User should pass USER_SPECIFIED_VALUE as string or integer. 

  Once the PVCNAMES are replaced use following commands to apply the manifest files

  `kubectl -n opsmx-isd apply -f minio/`

  Once the above is command is executed pvc will be created and pod will be deployed and backup will be stored in that pvc .please wait 10sec and check     the logs the backup will be sucessfull.

  `kubectl -n opsmx-isd apply -f redis/`

  Once the above is command is pvc will be created and pod will be deployed and backup will be stored in that pvc executed please wait 10sec and check     the logs the backup will be sucessfull.

  `kubectl -n opsmx-isd apply -f postgres/`

  Once the above is command is executed pvc will be created and pod will be deployed and backup will be stored in that pvcplease wait 10sec and check the   logs the backup will be sucessfull.

  Plese verify with the following command if the pvcs created or not

  `kubectl -n opsmx-isd get pvc | grep pvc-`

  Once the backup is sucessfull proceed for the Db Migration.
 
4. ISD 3.11 involves database changes so below are the steps followed to upgrade the DB.
    
   Please login to the spinnaker and create a new pipeline (i.e dbmigration) in opsmx-gitops.

   Copy the raw content of the dbmigration and Click the on edit as json and update the json file and save the pipeline.

   User just need to pass the source(i.e to which ISD version to be upgraded ) and namespace and run the pipeline.
   
   Once the job is sucessfull proceed for next step.


5. To upgrade from current ISD version(i.e 3.10) to desired (i.e 3.11) User need to create a new configmap 

   In the gitops-repo cloned to disk and edit upgrade/upgradecm.yaml. This should be updated with version of ISD (application version ), gitrepo,            srcbranch and user details.

   `kubectl -n opsmx-isd apply -f upgrade/upgradecm.yaml` # Edit namespace if changed from the default "opsmx-isd"

    `Once the job is sucessfull`

    A new branch(i.e branch will be the same as ISD version) will be created and all the manifest files will be uploaded in that repo.

    Compare the new deployment yamls with the previous version to check the differences. Reconcile any differences, raise a PR and merge the new versions     on to the default ("main") branch.

 6. In the next step user need to apply the apply the uploaded manifest files into the namespace

    Use below command to apply the manifest files

    `kubectl -n opsmx-isd apply -f reinstall.yaml`

    Check the status of the pods by executing this command:

    `kubectl -n opsmx-isd get po`

     Once all pods show "Running" or "Completed" status, wait for a couple of minutes and access the ingress

     `kubectl -n opsmx-isd get ing`

7. Login to the the ISD URL and confirm that the upgrade is completed by checking the version at the top-right corner or lower right corner.

8. Go to all the screens in ISD UI and verify everything works fine or not.

