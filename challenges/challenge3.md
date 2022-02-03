# Introduction
Metaverse is booming and it is about to create a whole new interaction paradim. 
Since Metaverse is relying on 3d assets that usually take a bit of time to model manually, we belive that using data to programatically create 3d assets will benefit whole industry going forward. Using AI to programatic  create 3d asset is not a new concept and there has been a vast number of R&D papers looking at applying AI for various use cases spaning across animations, shaders, sculping or converting 2d images to 3d assets.

# Challenge
In challenge 3 of this hackathon, we would like for you to create 3d track programatically using BPY.
We will provide you with API for one of the tracks and we need to create a script that can turn x,y and z cordinates into awesome track.

![](../img/Track.png?raw=true)

# Data Set Notes
As you know Formual 1 races take place in different cities. Therefore you need to bare in mind that your script need to be scalable so that we can apply it to other track coordiantes.


## Blender Python API
Blender python sits at the heart of this challenge, feel free to explore this page to get more info regarding how it works
https://docs.blender.org/api/current/info_quickstart.html

Blender Python API features:
1. Edit any data the user interface can (Scenes, Meshes, Particles etc.).
2. Modify user preferences, keymaps and themes.
3. Run tools with own settings.
4. Create user interface elements such as menus, headers and panels.
5. Create new tools.
6. Create interactive tools.
7. Create new rendering engines that integrate with Blender.
8. Subscribe to changes to data and it’s properties.
9. Define new settings in existing Blender data.
10. Draw in the 3D Viewport using Python.

# Scoring
The following things will be taken into consideration when doing an independent evaluation of challenge 3:
1. BPY script running on our local machine
2. BPY script generating track from different coordantes that the ones that we provided on the day
3. BPY procedurally generating surface material
4. Wow factor - this is a bit subjective hence we will get our internal teams to vote on the favourite track/tracks. To help us sell your vision submit a video that shows off your work.

### Code Presentation and Readability

Clean code is code that is easy to understand and easy to change.

It's writing code that is:
- Readable
- Understandable
- Maintainable
- Extendable

We recommend adhering to the [PEP 8 -- Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/#naming-conventions), but if the code is readable and easily understandable we will accept it as well as any other code adhering to the standard.

### Using OCI

We suggest looking into Oracle Cloud Infrastructure as a possible services that could be useful during development / submission.

Here are some ideas for reference:
- [Running Blender on OCI (GUI)](https://www.youtube.com/watch?v=amqxaw2Ujn4&ab_channel=OracleDevelopers)
- [Running Blender on OCI (Headless)](https://jeffmdavies.medium.com/blender-2-83-on-oracle-cloud-infrastructure-80ecfcb2ce4e)

## License
Copyright (c) 2022 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](LICENSE) for more details.

ORACLE AND ITS AFFILIATES DO NOT PROVIDE ANY WARRANTY WHATSOEVER, EXPRESS OR IMPLIED, FOR ANY SOFTWARE, MATERIAL OR CONTENT OF ANY KIND CONTAINED OR PRODUCED WITHIN THIS REPOSITORY, AND IN PARTICULAR SPECIFICALLY DISCLAIM ANY AND ALL IMPLIED WARRANTIES OF TITLE, NON-INFRINGEMENT, MERCHANTABILITY, AND FITNESS FOR A PARTICULAR PURPOSE. FURTHERMORE, ORACLE AND ITS AFFILIATES DO NOT REPRESENT THAT ANY CUSTOMARY SECURITY REVIEW HAS BEEN PERFORMED WITH RESPECT TO ANY SOFTWARE, MATERIAL OR CONTENT CONTAINED OR PRODUCED WITHIN THIS REPOSITORY. IN ADDITION, AND WITHOUT LIMITING THE FOREGOING, THIRD PARTIES MAY HAVE POSTED SOFTWARE, MATERIAL OR CONTENT TO THIS REPOSITORY WITHOUT ANY REVIEW. USE AT YOUR OWN RISK. 