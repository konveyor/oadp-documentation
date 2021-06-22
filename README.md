# OADP Documentation
Documentation for OpenShift API for Data Protection Operator

[Documentation (development version)](https://oadp.konveyor.io/) 

## Adding a New Page:
### Install Hugo:
This documentation site is generated using [Hugo.](https://gohugo.io/)

To add additional documentation to this site, you must first install Hugo.

Click [here](https://gohugo.io/getting-started/installing) for install 
instructions.

### Create a New Page:
Once Hugo is installed, use the following command:

`hugo new content/<CATEGORY>/<FILE.md>`

Make sure `draft` is set to `false`. 

### Start the Server
To start the Hugo server, run the command:

`hugo server`

## Adding a Page to the Navigation
The navigation menu is located at `data/menu/main.yml`.
To add an additional page, enter a chosen name under the appropriate 
section, and the link to that page. 

An example of the Overview page:
```
  - name: Getting Started
    sub:
      - name: Overview
        ref: "/Getting Started/overview"
```
