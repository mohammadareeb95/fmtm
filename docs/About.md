![](images/hot_logo.png)

# Field Mapping Tasking Manager (FMTM)

## A project to provide tools for Open Mapping campaigns

While we have pretty good field mapping applications, we don’t have great tools to coordinate field mapping. However, we have most of the elements needed to create a field mapping-oriented version of the HOT Tasking Manager, which allows people to select specific areas and complete tasks in a coordinated, organized fashion.

It’s already possible to implement a sort of Field Mapping Tasking Manager workflow using existing tools (mostly based on ODK, particularly the new "select from map" functionality), but it’s pretty labor-intensive and tricky. We’re working on some automation to ease the pain points, which moves us closer to a fully-fledged Field Mapping Tasking Manager (FMTM).

[Background and description of the project and idea are here: please have a look at this blog if you haven't yet!](https://www.hotosm.org/updates/field-mapping-is-the-future-a-tasking-manager-workflow-using-odk/)

# How to contribute

👍🎉We are actively looking for contributors to this project- from design, user testing, and both front and backend developers. Please take a look at our [contributor guidance](https://github.com/hotosm/fmtm/blob/main/CONTRIBUTING.md) for more details! Reach out to us if any questions! 👍🎉

# Using OpenDataKit's Select From Map feature

As of mid-2022, ODK incorporates a new functionality, select from map, that allows field mappers to select an object from a map, view the existing attributes, and fill out a form adding new information and attributes to that object. For example, a mapper can approach a building, select that building from a map view within ODK on their mobile phone, and add the opening hours, number of floors, construction material, or any number of useful attributes in a well-structured questionnaire format

![ODK's select_one_from_file](images/ODK_Select_one_from_file_map_screenshot.jpg)

To prepare the appropriate map files for ODK, we are taking our inspiration from the [HOT Tasking Manager](https://tasks.hotosm.org/), which allows remote mappers to choose well-defined small "task" areas, ensuring full coverage of the project area and no unintended duplication of tasks.

![HOT Tasking Manager allows users to choose areas just for themselves](images/HOT_TM_task_selection_screenshot.jpg)

# Users

## Campaign managers

Campaign managers select an Area of Interest (AOI) and organize field mappers to go out and collect data. They need to:

- Select an AOI polygon by creating a GeoJSON or by tracing a polygon in a Web map
- Choose a task division scheme (number of features or area per task, and possible variations on what features to use as the preferred splitting lines)
- Provide specific instructions and guidance for field mappers on the project.
- Provide a URL to a mobile-friendly Web page where field mappers can, from their mobile phone, select a task that is not already "checked out" (or possibly simply allocate areas to the field mappers).
- See the status of tasks (open, "checked out", completed but not validated, requires to rework, validated, etc) in the Web browser on their computer

## Field mappers

Field mappers select (or are allocated) individual tasks within a project AOI and use ODK Collect to gather data in those areas. They need to:

- Visit a mobile-friendly Web page where they can see available tasks on a map
- Choose an area and launch ODK Collect with the form corresponding to their allocated area pre-loaded

## Validators

Validators review the data collected by field mappers and assess its quality. If the data is good, the validators merge the portion of the data that belongs in OpenStreetMap to OSM. If it requires more work, the validators either fix it themselves (for minor stuff like spelling or capitalization mistakes that don't seem to be systematic) or inform the field mappers that they need to fix it. They need to:

- Access completed data sets of "submissions" as Comma Separated Values and/or OSM XML so that they can review them.
- Mark areas as validated or requiring rework
- Communicate with field mappers if rework is necessary
- Merge good-quality data into OSM (probably from JOSM).
- Mark areas as completed and merged.

# Info for developers

The basic setup here is:

## ODK Collect

A mobile data collection tool that functions on almost all Android phones. Field mappers use ODK Collect to select features such as buildings or amenities, and fill out forms with survey questions to collect attributes or data about those features (normally at least some of these attributes are intended to become OSM tags associated with those features).

The ODK Collect app connects to a back-end server (in this case ODK Central), which provides the features to be mapped and the survey form definitions.

## ODK Central server

An ODK Central server that functions as the back end for the field data collectors' ODK Collect apps on their Android phones. Devs must have access to an ODK Central server with a username and password granting admin credentials.

[Here are the instructions for setting up an ODK Central server on Digital Ocean](https://docs.getodk.org/central-install-digital-ocean/) (it's very similar on AWS or whatever)

## Field Mapping Tasking Manager Web App

The FMTM web app is a Python/Flask/Leaflet app that serves as a front end for the ODK Central server, using the [ODK Central API](https://odkcentral.docs.apiary.io/#) to allocate specific areas/features to individual mappers, and receive their data submissions.

### Manager Web Interface (with PC browser-friendlymap view)

A computer-screen-optimized web app that allows Campaign Managers to:

- Select AOIs
- Choose task-splitting schemes
- Provide instructions and guidance specific to the project
- View areas that are at various stages of completion
- Provide a project-specific URL that field mappers can access from their mobile phones to select and map tasks.

### FMTM back end

A back end that converts the project parameters entered by the Campaign Manager in the Manager Web Interface into a corresponding ODK Central project. Its functions include:

- Convert the AOI into a bounding box and corresponding Overpass API query
- Download (using the Overpass API) the OSM features that will be mapped in that bounding box (buildings and/or amenities) as well as the OSM line features that will be used as cutlines to subdivide the area
- Trim the features within the bounding box but outside the AOI polygon
- Convert the polygon features into centroid points (needed because ODK select from map doesn't yet deal with polygons; this is likely to change in the future but for now we'll work with points only)
- Use line features as cutlines to create individual tasks (squares don't make sense for field mapping, neighborhoods delineated by large roads, watercourses, and railways do)
- Split the AOI into those tasks based on parameters set in the Manager Web Interface (number of features or area per task, splitting strategy, etc).
- Use the ODK Central API to create, on the associated ODK Central server:
  - A project for the whole AOI
  - One survey form for each split task (neighborhood)
    - This might require modifying the xlsforms (to update the version ID of the forms and change the name of the geography file being referred to). This is pretty straightforward using [OpenPyXL](https://openpyxl.readthedocs.io/en/stable/), though we have to be careful to keep the location within the spreadsheet of these two items consistent.
  - GeoJSON feature collections for each form (the buildings/amenities or whatever)
  - An App User for each form, which in turn corresponds to a single task. When the ODK Collect app on a user's phone is configured to function as that App User, they have access to _only_ the form and features/area of that task.
  - A set of QR Codes and/or configuration files/strings for ODK Collect, one for each App User

### Field Mapper Web Interface (with mobile-friendly map view)

- Ideally with a link that opens ODK Collect directly from the browser, but if that's hard, the fallback is downloading a QR code and importing it into ODK Collect.
