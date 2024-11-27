# Wazi Deploy Recipes

Collection of snippets and recipes when using Wazi Deploy

## Use `job_submit` to generate a Job based on a jinja2 template for a set of artifacts

The [custom_jcl_replacing.jcl.j2](templates/custom_jcl_replacing.jcl.j2) template outlines how to generate a single job with a step for each artifact that is matching the selection criteria.

Here is a sample activity configuration as a sample for your deployment method:

```yaml
  - name: CUST_JCL
    description: |
      This will invoke the custom updates
    actions:
       - name: JCL
         states:
            - UNDEFINED
         - name: CUSTOM_REPLACING_LOGIC
           description: |
             This step is using the job_submit building block to 
             generate a job that can invoke for instance an existing 
             replacing/substitution logic for JCL members.
           is_artifact: True
           types: 
            - name: "JCL"
           when:
            - wd_query[[ length ( current_plan_step.artifacts) ]] != 0
           properties:
           - key: template
             value: job_submit
           - key: var_job_submit
             value: custom_jcl_replacing  
```

Your environment file contains the configuration for the `job_submit` building block

```
  custom_jcl_replacing:
   src: "/u/dbehm/wazi-deploy/cfg/templates/custom_jcl_replacing.jcl.j2"
   use_template: True
   dest: "custom_jcl_replacing.jcl"
```

## Use `job_submit` to generate a Job based on a jinja2 template for a each artifact

The [custom_jcl_replacing_loop.jcl.j2](templates/custom_jcl_replacing_loop.jcl.j2) template outlines how to generate a job for each artifact that is matching the selection criteria.

Here is a sample activity configuration as a sample for your deployment method. The `loop_query` selects the current_plan_step_artifacts and pass the list to `single_variable`.  Based on the loop condition, this step will be invoked x-times:


```
         - name: CUSTOM_REPLACING_LOOP
           is_artifact: True
           types: 
            - name: "JCL"
           loop:
            loop_query: current_plan_step.artifacts
            loop_var: single_artifact
           when:
            - wd_query[[ length ( current_plan_step.artifacts) ]] != 0
           properties:
           - key: template
             value: job_submit
           - key: var_job_submit
             value: custom_jcl_replacing_loop
```

Your environment file contains the configuration for the `job_submit` building block. 

```
  custom_jcl_replacing_loop:
   src: "/u/dbehm/wazi-deploy/cfg/templates/custom_jcl_replacing_loop.jcl.j2"
   use_template: True
   dest: "custom_jcl_replacing_loop.jcl"
```