---
layout: default
title: Sitecore Forms Customization
nav_order: 2
---

# Configuration
{: .no_toc }

My goal in this task was to create a custom layout for the sitecore forms as you can see in the picture
{: .fs-5 .fw-300 }


![](/SitecoreBlog/assets/images/529dddab-8f57-4169-864b-70804bb6ddd2.png)


## Table of contents
{: .no_toc}

1. TOC
{:toc}

---

## How Sitecore forms are working

The main Idea is that you drag a form element from the left panel and drop it in the middle to create a form field. then you attach this form to a content item to render it, finally when the user submit the form you can configure some action, in this case it will be send email. 

<img style="width:400px;margin: auto;display: block;" 
     src="/SitecoreBlog/assets/images/3d22fd22-f60c-40e0-95d5-d64515ab996a.png">

Take a quick look at [sitecore article](https://doc.sitecore.com/developers/91/sitecore-experience-manager/en/walkthrough--creating-a-custom-submit-action.html), we will discuss more details here 

![](/SitecoreBlog/assets/images/4f1bf06d-9acf-43b2-bba1-24015a89cb32.png)

To create new control you need to:

1.  Create a **form element** item under `/sitecore/system/Settings/Forms/Field Types`
    -  As soon as you do that you will be able to see the new form field you can drag it to the form but it will not do anything, as Sitecore doesn’t know how to render it yet
    -  This is the main item that tell sitecore how this field should be rendered and how it behave  
1.  Create a **view cshtml** on the folder `{sitecorepath}\Views\FormBuilder\FieldTemplates`
    -  Under the View Path field of the form element item, enter the cshtml file that you just created
1.  Create a **Model** (find more details in the link above)
Write the full name of the class in the Model Type Field 
1.  Specify the **template** in the field
    -  Under the `Field Template` field specify the template that you want to use 
    - `WARNING`{: .label .label-yellow } If you created your own template make sure that it has a standard values, otherwise you will not be able to see anything 
in the form when you drag the control and there is no warnings or errors happening, it simply will not work.   
1.  `Property Editor` field will control the properties of that form element
When you select the form, a list of properties will be shown in the left panel, that is configured in the core db. you can see the Details section and the Styling and so on.

## How to debug it

Most of the time you will be in a situation where it just doesn’t work and there is no exceptions in the log, so you will have absolutely no clue on what’s going on. 

You have to start dissemble some dlls and start replacing the default Sitecore config and debug it. 

`\App_Config\Sitecore\ExperienceForms` => `Sitecore.ExperienceForms.Pipelines.Client.config`

As a start there is controller called `FormBuilderController` any form submission will go there first before getting redirected to internal pipelines. to change it you need to also replace `InitializeRoutes` to add your own route and make sure to change the namespace as well in the route.

while working you might need to change some databinders or some other pipelines code. 

```scss
Make sure to have time for this in your estimation
```

## Technical Challenges
-  Multi-Page form is not working for some reason
    -  for some wired reason the first page will be rendered fine but if you moved to the next page, the form will lose the context and all the headers and footers will disappear the style will be messed up.
-  There is a very wired behavior in Sitecore if you changed the Submit button css class it will throw a BE null pointer exception, to fix it I had to customize the `NavigationDataModelBinder` & `SetModelBinders` and remove some if statements from the first class. `navigationData` was null.
-  If you have a custom action inheriting from `SubmitActionBase` but you don’t have any parameters your `Execute` method will never get called, you have to override the `ExecuteAction` method, it’s another bug in Sitecore.
-  Sitecore Form uses `data-sc-field-name` html attribute and `data-sc-field-key` to fill in the model class `FormSubmitContext` in other words to identify the field and get the value and fill the Model from the form data (request data).

