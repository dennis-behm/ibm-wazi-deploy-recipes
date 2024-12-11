# Wazi Deploy Recipes

Collection of snippets and recipes when using Wazi Deploy

## Use `job_submit` to generate a Job based on a jinja2 template for a set of artifacts

The [custom_jcl_replacing.jcl.j2](templates/custom_jcl_replacing.jcl.j2) template outlines how to generate a single job with a step for each artifact that is matching the selection criteria.

Here is a sample activity configuration as a sample for your deployment method:

```yaml
  - name: CUST_JCL
    description: |
      This will invoke the custom updates of JCL
    actions:
       - name: JCL_SUBSTITUTION
         states:
            - UNDEFINED
         steps:
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

```yaml
  custom_jcl_replacing:
   src: "/u/dbehm/wazi-deploy/cfg/templates/custom_jcl_replacing.jcl.j2"
   use_template: True
   dest: "custom_jcl_replacing.jcl"
```

## Use `job_submit` to generate a Job based on a jinja2 template for a each artifact

The [custom_jcl_replacing_loop.jcl.j2](templates/custom_jcl_replacing_loop.jcl.j2) template outlines how to generate a job for each artifact that is matching the selection criteria.

It leverages the [conditional keyword loop](https://www.ibm.com/docs/en/developer-for-zos/17.0?topic=reference-syntax-conditional-deployment#wd_conditional_depl__title__3) that got introduced with Wazi Deploy 3.0.3

Here is a sample activity configuration as a sample for your deployment method. The `loop_query` selects the current_plan_step_artifacts and passes the list to `single_variable` in the environment dict. Based on the loop condition, this step will be invoked x-times:

```yaml
  - name: CUST_JCL
    description: |
      This will invoke the custom updates of JCL
    actions:
       - name: JCL_SUBSTITUTION
         states:
            - UNDEFINED
         steps:
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

```yaml
  custom_jcl_replacing_loop:
   src: "/u/dbehm/wazi-deploy/cfg/templates/custom_jcl_replacing_loop.jcl.j2"
   use_template: True
   dest: "custom_jcl_replacing_loop.jcl"
```

## Use a generic action block to process multiple types with a jinja2 template

There may be situations, where you have multiple variations of JCL or other text files such as control cards that need to be customized to the target environment of the deployment environment. 
You already know that you need to process all in a very similar fashion. For instance by leveraging the job_submit with a jinja2 template.

Instead of repeating a similar block 10 or even more times, you can apply the following recipe and leverage a "generic" action block to dynamically configure the `job_submit` step. 

The action is composed of 2 steps - reading a parameterized input configuration file; which is followed by the step using job_submit, combined with a loop on the level of the action.
In the deployment method, we leverage the `loop_query` to iterate over all the deployTypes:

```yaml
  - name: GENERIC_JCL_PROCESSING
    description: |
      This will invoke the custom updates for multiple outputs
    actions:
       - name: GENERIC_TEXT_CUSTUMIZATION
         description: | 
           This action block show cases, how to loop on the action level 
           over multiple deployTypes and read dynamically 
           the configuration for a job_submit step.
           Imagine the JCLM1..JCLM5 to be different groups with different
           libraries, all requiring some post deployment processing
         states:
            - UNDEFINED
         is_artifact: true
         types: 
            - name: "JCLM1"
            - name: "JCLM2"
            - name: "JCLM3"
            - name: "JCLM4"
            - name: "JCLM5"
         loop:
           loop_query: current_plan_action.types
           loop_var: artifactType
         steps:
         - name: READ_GENERIC_CONFIGURATION
           short_name: GENERIC_CONFIG
           description: | 
             This step is reading a generic configuration file that includes
             references to the loop variable artifactType 
           is_artifact: true
           properties:
             - key: template
               value: include_vars
             - key: var_include_vars
               value: include_generic_jcl_vars
         - name: GENERIC_REPLACING_LOGIC
           short_name: GENERIC_REPLACE
           is_artifact: true
           description: |
             This step is using the job_submit building block to 
             generate a job that can invoke for instance an existing 
             replacing/substitution logic for the defined types.
           when:
            - wd_query[[ length ( current_plan_step.artifacts) ]] != 0
           properties:
           - key: template
             value: job_submit
           - key: var_job_submit
             value: generic_jcl_replacing
```

Your environment file contains the references to the parametrized yaml configuration.

```yaml
include_generic_jcl_vars:
 files_list:
 - "{{ cfg }}/vars/generic_jcl_replacing.yaml"
 files_must_exist: True
```

The sample `generic_jcl_replacing.yaml` is loaded dynamically, and configures the `src` and `dest` for each iteration of the loop.

It additionally show cases, how you can dynamically select additional configuration parameters for each iteration depending on the deploy type. In the below samples, there are srcPDS and targetPDS datasets configured for the generic jcl_replacing jinja2 template.

```yaml
  jcl_types:
   - type: "JCLM1"
     srcPDS: "{{ hlq }}.TEMP.JCLM1"
     targetPDS: "{{ hlq }}.JCLM1"
   - type: "JCLM2"
     srcPDS: "{{ hlq }}.TEMP.JCLM2"
     targetPDS: "{{ hlq }}.JCLM2"
   - type: "JCLM3"
     srcPDS: "{{ hlq }}.TEMP.JCLM3"
     targetPDS: "{{ hlq }}.JCLM3"
   - type: "JCLM4"
     srcPDS: "{{ hlq }}.TEMP.JCLM4"
     targetPDS: "{{ hlq }}.JCLM4"    
   - type: "JCLM5"
     srcPDS: "{{ hlq }}.TEMP.JCLM5"
     targetPDS: "{{ hlq }}.JCLM5"
      
  generic_jcl_replacing:
   src: "{{ cfg  }}/templates/custom_jcl_replacing_generic.j2"
   use_template: True
   dest: generic_jcl_replacing_{{ artifactType.name | lower }}.jcl
   srcPDS: "{{ jcl_types | selectattr( 'type', 'equalto', artifactType.name ) | map(attribute='srcPDS') | list | first }}"
   targetPDS: "{{ jcl_types | selectattr( 'type', 'equalto', artifactType.name ) | map(attribute='targetPDS') | list | first }}"
```

You find the template that references srcPDS and targetPDS in [templates/custom_jcl_replacing_generic.jcl.j2](templates/custom_jcl_replacing_generic.jcl.j2).