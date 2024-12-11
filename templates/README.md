# Template details

This readme provides further details on the jinja2 templates in this folder.

## custom_jcl_for_entire_set template

The [custom_jcl_replacing.jcl.j2](custom_jcl_replacing.jcl.j2) processes the `step.artifacts` as mentioned in the [Wazi Deploy documentation](https://www.ibm.com/docs/en/developer-for-zos/17.0?topic=translators-coding-variables-in-jinja2-templates).

It can be used to integrate existing JCL substitution procedures. The template today show cases to generate a DD  with some instream data about each artifact, it iterates of the entire set.

<details>
  <summary>Sample output of the generated JCL</summary>

```
//CUSTJCL JOB 'WD-CJCL',MSGLEVEL=(1,1),MSGCLASS=R,NOTIFY=&SYSUID
//*ROUTE PRINT @JCLPRINT@
//*JOBLIB DD DISP=SHR,
//* DSN=TODO.SDSNEXIT
//* DD DISP=SHR,
//* DSN=TODO.SDSNLOAD
//*******************************************
//*
//* Notes:
//*
//* This Jinja2 templates iterate over the step.artifacts and allows
//* to put together a custom JCL for the package contents, that are 
//* referenced in this deployment step.
//*
//* This template is processed once for all associated types. It does
//* not submit a JCL per member (the submit_job building block is not 
//* artifact "sensitive")
//*
//* In this sample we are printing everything to the instream dd for IEFBR14
//*
//* Author DBEHM
//*******************************************
//* CUSTOMIZATION JOB
//*******************************************
//**BEGIN
//JCLCUST EXEC PGM=IEFBR14,DYNAMNBR=20,COND=(4,LT)
//SYSTSPRT DD SYSOUT=*
//SYSPRINT DD SYSOUT=*
//SYSUDUMP DD SYSOUT=*
//SYSIN DD DUMMY
//SYSIN DD *
//EBUD0RUN DD *           <--------- first member
Element name: EBUD0RUN

Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RUN.JCL
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RUN 
//*
//EBUD0RU1 DD *           <--------- second member
Element name: EBUD0RU1

Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU1.JCL
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU1 
//*
//*
```

</details>

## custom_jcl_for_each_artifact template

The [custom_jcl_replacing_loop.jcl.j2](custom_jcl_replacing_loop.jcl.j2)  processes the `single_artifact` variable that is extracted by the conditional deployment `loop` condition. 

It can be used to integrate existing JCL substitution procedures and run an job for each artifact. The template show cases to generate the DD  with some instream data about the  artifact that is passed in. It submits a job for each artifact!


<details>
  <summary>Sample output of the generated JCL</summary>

```
//CUSTJCL JOB 'WD-CJCL',MSGLEVEL=(1,1),MSGCLASS=R,NOTIFY=&SYSUID
//*ROUTE PRINT @JCLPRINT@
//*JOBLIB DD DISP=SHR,
//* DSN=TODO.SDSNEXIT
//* DD DISP=SHR,
//* DSN=TODO.SDSNLOAD
//*******************************************
//*
//* Notes:
//*
//* This Jinja2 templates iterate over the variable that is passed in
//* over the loop conditions
//*
//* This template is processed for each artifact. It does
//*  submit a JCL per member
//*
//* In this sample we are printing everything to the instream dd for IEFBR14
//*
//* Author DBEHM
//*******************************************
//* CUSTOMIZATION JOB
//*******************************************
//**BEGIN
//JCLCUST EXEC PGM=IEFBR14,DYNAMNBR=20,COND=(4,LT)
//SYSTSPRT DD SYSOUT=*
//SYSPRINT DD SYSOUT=*
//SYSUDUMP DD SYSOUT=*
//SYSIN DD DUMMY
//SYSIN DD *
//EBUD0RUN DD *
Element name: EBUD0RUN   <--------- for each artifact

Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RUN.JCL
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RUN 
//*
//*
```

</details>

## custom_jcl_for_artifact_set with dynamic input

The [custom_jcl_replacing_generic.jcl.j2](custom_jcl_replacing_generic.jcl.j2) is similar to [custom_jcl_replacing.jcl.j2](custom_jcl_replacing.jcl.j2), but showcases how to reference some input variables, that got passed in. 

It is related to the [use-a-generic-action-block-to-process-multiple-types-with-a-jinja2-template](../README.md#use-a-generic-action-block-to-process-multiple-types-with-a-jinja2-template) recipe.

<details>
  <summary>Sample output of the generated JCL</summary>

```
//CUSTJCL JOB 'WD-CJCL',MSGLEVEL=(1,1),MSGCLASS=R,NOTIFY=&SYSUID
//*ROUTE PRINT @JCLPRINT@
//*JOBLIB DD DISP=SHR,
//* DSN=TODO.SDSNEXIT
//* DD DISP=SHR,
//* DSN=TODO.SDSNLOAD
//*******************************************
//*
//* Notes:
//*
//* This Jinja2 templates iterate over the step.artifacts and allows
//* to put together a custom JCL for the package contents, that are 
//* referenced in this deployment step.
//*
//* This template is processed once for all associated types. It does
//* not submit a JCL per member (the submit_job building block is not 
//* artifact "sensitive")
//*
//* In this sample we are printing everything to the instream dd for IEFBR14
//*
//* Author DBEHM
//*******************************************
//* CUSTOMIZATION JOB
//*******************************************
//**BEGIN
//JCLCUST EXEC PGM=IEFBR14,DYNAMNBR=20,COND=(4,LT)
//SYSTSPRT DD SYSOUT=*
//SYSPRINT DD SYSOUT=*
//SYSUDUMP DD SYSOUT=*
//SYSIN DD DUMMY
//SYSIN DD *
//*IN    DD DSN=DBEHM.WDEPLOY.RETCAL.INT2.TEMP.JCLM1,DISP=SHR
//*OUT   DD DSN=DBEHM.WDEPLOY.RETCAL.INT2.JCLM1,DISP=SHR
//EBUD0RU3 DD *
Element name: EBUD0RU3
Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU3.JCLM3
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU3 
//*
//EBUD0RU2 DD *
Element name: EBUD0RU2
Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU2.JCLM2
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU2 
//*
//EBUD0RU5 DD *
Element name: EBUD0RU5
Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU5.JCLM5
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU5 
//*
//EBUD0RU4 DD *
Element name: EBUD0RU4
Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU4.JCLM4
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU4 
//*
//EBUD0RU1 DD *
Element name: EBUD0RU1
Element properties:
   path: GITLAB.RETIREME.ETR.JCL/EBUD0RU1.JCLM1
Element hash: 5f58530969d155d3f239a5fddace93f32383e21040b633681456588b330c8473
end EBUD0RU1 
//*
//*
```

</details>