# Migrate from lumber v2 to v3

## Foreword

[Lumber](https://github.com/ForestAdmin/lumber) is the CLI tool used to generate your admin backend and install Forest Admin. Significant [changes have occurred between Lumber v2 and v3](https://github.com/ForestAdmin/lumber/blob/devel/CHANGELOG.md#release-300-beta0---2019-11-19) including the generation of the `forest` and `routes` folder \(as shown in the page listed below\) and more complete logs.

[Overview of changes induced by Lumber v3](Migrate%20from%20lumber%20v2%20to%20v3/Overview%20of%20changes%20induced%20by%20Lumber%20v3.md)

While you can perfectly keep working with your lumber v2 admin backend, you may want to upgrade to create a new one to benefit from those changes.

{% hint style="info" %}
The present note aims at **providing guidance regarding the switch from Lumber v2 to Lumber v3**. Without losing the customisation you made to your admin backend or to your layout.
{% endhint %}

For the context of this note, we consider the user has been following these steps prior to migrating from v2 to v3:

* Created a project in Forest Admin \(called **project A** hereafter\)
* Generated an admin backend using Lumber v2 to create this project
* Customized the admin backend \(by adding smart fields and smart actions for example\)
* Customized the UI layout in Forest Admin \(hide/show relevant collections and fields, apply widgets to specific fields, create summary views, etc\)

Now to migrate from the lumber V2 admin backend to a lumber V3 one for project A, you need to perform the following:

* Step 1: generate your lumber v3 admin backend
* Step 2: Integrate this new app to project A on Forest Admin
* Step 3: Migrate all the customisation you made to your admin backend and Forest Admin UI
* Step 4: Make the new lumber v3 app the default admin backend of project A

## Step 1: Generating your lumber v3 admin backend with npm

First you need to create a new project \(hereafter **project B**\) within Forest Admin to generate an admin backend with the v3 of Lumber. This project will **only be used to generate your admin backend** and will be discarded afterwards.

The database used during the creation of project B should be the same database as the one associated with the admin backend of project A. This ensures that the models generated in your admin backend will be the same which will simplify the migration.

{% hint style="info" %}
The documentation relating to the creation of a new project can be found here ⇒ [https://docs.forestadmin.com/documentation/getting-started/installation](https://docs.forestadmin.com/documentation/getting-started/installation).
{% endhint %}

Once you have followed the installation steps with npm, you have a new admin backend that includes the following in a `.env` file:

```text
APPLICATION_PORT=3310

CORS_ORIGINS=

DATABASE_URL=postgres://forest:secret@localhost:5451/custom-db-movies
DATABASE_SCHEMA=public
DATABASE_SSL=false

FOREST_AUTH_SECRET=498a039f0249c5c1439c84a1a4444a371312fd59cda1a9f94921c1227e7b3be7ff65cfdb9bf5f47f2a26f085862991
FOREST_ENV_SECRET=123297e62f12717c28c82e93ac7433c2466d6c9e6d92ec0f834f68450160296c
```

The `APPLICATION_PORT` corresponds to the port on which the admin backend runs.

The **database information** is the one you provided upon generation of the project. Here my credentials are:

`database type`: postgresql

`user`: forest

`password`: secret

`host`: localhost

`port`: 5451

`name`: custom-db-movies

`schema`: public

`SSL encryption`: No

The `FOREST_AUTH_SECRET` is a random string that can be defined by the user. It allows for the authentication of the requests sent to retrieve the data.

The `FOREST_ENV_SECRET` allows for the authentication with the Forest Admin servers to retrieve the configuration of the layout for the development environment created along with the new project.

You needed to create a new project to generate your admin backend. But you will not use the project created. [You can delete it](http://g.recordit.co/mri73U3Ga0.gif).

{% hint style="info" %}
At this point you have a new lumber v3 application \(your lumber v3 admin backend\) configured with the credentials of the database used for project A.
{% endhint %}

{% hint style="info" %}
You still need to \(i\) integrate the new app to project A and \(ii\) migrate the customization performed from your lumber v2 admin backend to your new lumber v3 admin backend and \(iii\) make the new lumber v3 admin backend the default admin backend of project A
{% endhint %}

## Step 2: Integrating the new app to your existing project

To connect your lumber v3 admin backend to project A, the first step would be to create a new environment within project A.

{% hint style="info" %}
You will find here the documentation to create a new environment ⇒ [https://docs.forestadmin.com/documentation/reference-guide/how-it-works/environments\#creating-a-new-environment](https://docs.forestadmin.com/documentation/reference-guide/how-it-works/environments#creating-a-new-environment)
{% endhint %}

The application url you need to enter will be the application url of your lumber v3 app \(in the present example `http://localhost:3310`\).

As show in the documentation provided above, you will be given a `FOREST_ENV_SECRET` specific to the new environment upon its creation.

You need to use this value to set the `FOREST_ENV_SECRET` variable in the `.env` file of your lumber v3 admin backend.

You need to ensure that your app is running to validate the creation of the environment.

Once the environment is successfully created and connected to your lumber v3 admin backend, you can browse through your data and perform native actions in this new environment. However you **still need to migrate all the customization you had previously made both to your admin backend and UI** so your new environment is identical to your previous one.

## Step 3: Migrating the customisation you performed

### Migrating the customisation of your admin backend

If you had added customised your lumber v2 admin backend, you will want to migrate all the work done to the new admin backend. Customisation can mean adding smart features \(smart actions, smart fields, smart collections\) or changing the default configuration of your models for example.

While most of the code will remain the same and can be transposed from one app to the other, a few syntax changes need to be taken into consideration to ensure the code will run smoothly in lumber v3. These can be found in the following note:

[syntax changes from lumber v2 to lumber v3](Migrate%20from%20lumber%20v2%20to%20v3/syntax%20changes%20from%20lumber%20v2%20to%20lumber%20v3.md)

{% hint style="info" %}
**NB**: In addition, depending on the `forest-express-sequelize` version your admin backend was running with, you may need to take a look at the breaking changes between version as the new admin backend created will include the latest version of the package \(currently v5\).
{% endhint %}

{% hint style="info" %}
You can find here the documentation re. breaking changes from v4 to v5 ⇒ [https://docs.forestadmin.com/documentation/how-tos/upgrade-to-v5](https://docs.forestadmin.com/documentation/how-tos/upgrade-to-v5)
{% endhint %}

{% hint style="info" %}
At this point you have a running lumber v3 admin backend that includes all of the customisation you need for your admin panel to be operational in Forest Admin. And the corresponding environment accessible within your project. But **this new environment does not yet include the UI configuration you had already set up** for your previous environment.
{% endhint %}

### Copy the existing configuration of your UI layout

The UI configuration in Forest Admin is environment-specific. Therefore the environment you created to integrate your lumber v3 admin backend to your project does not include the customisation you had made to your other previous environments.

**You will need to copy the layout of the environment that has been previously properly configured.**

The first step would be to ensure that your schema is the same in your lumber v2 admin backend as it is in your new lumber v3 one. It is necessary that all your collection, field, action, etc match so that the configuration is properly applied.

To do so you need to compare the files `.forestadmin-schema.json` from both app repositories. As this file is the one that will be used by the Forest Admin server to apply the UI configuration, it needs to be identical to copy the layout entirely.

{% hint style="info" %}
If the schemas are the same, then you can proceed to copying the layout from your previous environment to the new one. The documentation for this can be found here ⇒ [https://docs.forestadmin.com/documentation/reference-guide/project-settings/environments-tab\#copying-a-layout-configuration](https://docs.forestadmin.com/documentation/reference-guide/project-settings/environments-tab#copying-a-layout-configuration).
{% endhint %}

## Step 4: Make the environment with the lumber v3 admin backend your default one

To complete the transition from your Lumber v2 admin backend to your lumber v3, you need to make your new environment \(linked to your lumber v3 admin backend\) the default environment of your project.

You can do so here in your project settings on the top right of your screen ⇒ general tab ⇒ default environment dropdown.

Once this is done you can delete the old environment by going to project settings ⇒ environments tab ⇒ delete environment \([gif here](https://recordit.co/tEMBJfSi1p/gif/notify)\).

{% hint style="info" %}
NB: If you had a production environment, you will need to deploy your lumber v3 admin backend in the same manner that you had deployed your lumber v2 admin backend in a new environment or erase your existing app directly. See how to do it in Heroku in the page below.
{% endhint %}

[Replace running app by app from another git repo on heroku](Migrate%20from%20lumber%20v2%20to%20v3/Replace%20running%20app%20by%20app%20from%20another%20git%20repo%20o.md)

