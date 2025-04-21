# Databricks Asset Bundle
Will demonstrate how to deploy and manage your assets efficiently

First step , I am setting up Databricks project in VS Code using Databricks extensions

Already I setup databricks plugin with VSCode


![image](https://github.com/user-attachments/assets/d19179dd-fc3b-44b8-bf3d-47e8137a72de)

Create PAT and update

![image](https://github.com/user-attachments/assets/bf8d617d-dcc3-4357-89ce-7a82609b14b0)


![image](https://github.com/user-attachments/assets/ac4dc77d-55d2-4b03-949b-928462e38fd7)

Validate bundle

![image](https://github.com/user-attachments/assets/ba67c2f6-55d4-47e6-9f08-d3c1a76f1156)

If you open workspace, you cannot see any files under DEV

![image](https://github.com/user-attachments/assets/0ec50eb2-b333-4bd2-8af3-31913111fbcf)



![image](https://github.com/user-attachments/assets/1d5ac4f6-50f4-4567-9736-d2d6ec4def47)

You can see Terraform files created in vs code after running 

databricks bundle deploy -t dev

![image](https://github.com/user-attachments/assets/ea46a609-0479-47bf-9971-5f2377c4facd)



And new files are deployed in Databricks workspace under Dev folder

![image](https://github.com/user-attachments/assets/c9b15e7d-d216-4e94-b306-77a2463a4cff)


Delta pipeline code in the vs code also copied to Databricks workspace

![image](https://github.com/user-attachments/assets/99a60fa0-1670-4cff-a189-b7ff30ed57cc)

![image](https://github.com/user-attachments/assets/c29e7c60-76d9-487d-9340-a1ccc20cd3e4)


Now I am running the job . Databricks automatically allocate job cluster and execute the job

![image](https://github.com/user-attachments/assets/b4bf5532-6475-4ea4-8c1e-9cd0a6eab481)

Just refer the job cluster mentioned in the yaml file

# The main job for cicdproject.
resources:
  jobs:
    cicdproject_job:
      name: cicdproject_job

      trigger:
        # Run this job every day, exactly one day from the last run; see https://docs.databricks.com/api/workspace/jobs/create#trigger
        periodic:
          interval: 1
          unit: DAYS

      email_notifications:
        on_failure:
          - bijumathewt@gmail.com

      tasks:
        - task_key: notebook_task
          job_cluster_key: job_cluster
          notebook_task:
            notebook_path: ../src/notebook.ipynb
        
        - task_key: refresh_pipeline
          depends_on:
            - task_key: notebook_task
          pipeline_task:
            pipeline_id: ${resources.pipelines.cicdproject_pipeline.id}
        
        - task_key: main_task
          depends_on:
            - task_key: refresh_pipeline
          job_cluster_key: job_cluster
          python_wheel_task:
            package_name: cicdproject
            entry_point: main
          libraries:
            # By default we just include the .whl file generated for the cicdproject package.
            # See https://docs.databricks.com/dev-tools/bundles/library-dependencies.html
            # for more information on how to add other libraries.
            - whl: ../dist/*.whl

      job_clusters:
        - job_cluster_key: job_cluster
          new_cluster:
            spark_version: 15.4.x-scala2.12
            node_type_id: Standard_D3_v2
            data_security_mode: SINGLE_USER
            autoscale:
                min_workers: 1
                max_workers: 4

I executed following DLT 

![image](https://github.com/user-attachments/assets/667d7212-cb2a-450b-9912-b61ca123b519)
