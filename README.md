<!-- This README file was based on Best-README-Template by Othneil Drew. Check https://github.com/othneildrew/Best-README-Template/tree/main to know more about the project. -->

<a id="readme-top"></a>

[![project_license][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<h1 align="center">Near-Field color maps</h1>

  <p align="center">
    This project provides near-field measured data of four commercial microwave lens horn antennas (from X- to Ka-band). It also provides a jupiter notebook script to create 2D color map of the measured data.
    <br />
    <br />
    <b>DISCLAIMER:</b> This project is part of an academic research about how to use an out-of-date robotic arm to measure the near-field of lens horn antennas. DOI number shall be updated as soon as publication become available.
    <br />
    <br />
    <a href="https://github.com/afnboss/nf-lens-horn-antenna"><strong>Explore the docs »</strong></a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About the project</a>
      <ul>
        <li><a href="#built-with">Built with</a></li>
      </ul>
    </li>
    <li>
      <a href="#how-to-use">How to use</a>
      <ul>
        <li><a href="#file-structure">File structure</a></li>
        <li><a href="#folders-and-subfolders">Folders and subfolders</a></li>
        <li><a href="#code">Code</a></li>
      </ul>
    </li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About the project

This notebook import and plot Near-Field (NF) measured data from XY planes acquired with a Vector Network Analyzer (VNA) using an out-of-date robotic arm. The functions defined in the project are:

- `importPlane`: import a single file from a folder, ignoring the head (skipRows=33);
  - Input: complete path of the data file;
  - Output: plane DataFrame;
- `importInfoPlane`: import a single file to extract informations about the scanning edges, distance between probe and AUT, and steps of each axis;
  - Input: complete path of the data file;
  - Output: a dictionary entry;
- `importMultPlanes`: import multiple files in a folder using importPlane and importInfoPlane;
  - Input: path containing all data file;
  - Output: a dictionary of DataFrames where the last entry is the information imported with importInfoPlane;
- `calcMag`: calculate the magnitude and phase of a sigle DataFrame. The normalized magnitude is calculated based on the higher value of the magnitude of the imported plane;
  - Requires execution of `importPlane` first;
  - Input: imported plane;
  - Output: add the magnitude (Mag), the normalized magnitude (Mag Norm), and the phase (Phase) at the end of the DataFrame columns;
- `calcMagMultPlanes`: calculate the magnitude, phase, and normalized magnitude using calcMag. It also calculate a normalized magnitude considering all planes;
  - Requires execution of `importMultPlanes` first;
  - Input: dictionary of planes and the column of the desired frequency;
  - Output: add Mag, Mag Norm, and Phase at the end of each DataFrame. It also calculate the normalized magnitude considering all planes (Mag Norm Planes). Returns the maximum and minumum values of the normalized magnitude of all planes;
- `plotMultPlanes`: plot all planes imported with importMultPlanes;
  - Requires execution of `calcMagMultPlanes` first;
  - Input: dictionary of planes and the column of the desired frequency;
  - Output: a list of bidimensional colormap plots;
- `plotSinglePlane`: used to export a single plotted plane as PNG. It also performs an interpolation of the center vertical and horizontal lines to provide an approximation of the beamwaist dimension;
  - Requires execution of `calcMagMultPlanes` first;
  - Input: dictionary of planes, the desired plane, and the column of the desired frequency;
  - Output: single plane plot with the size information of the beamwaist;
- `cutPlane`: plot the vertical (transverse) or the horizontal (longitudinal) cuts of the normalized magnitude of all planes;
  - Requires execution of `calcMagMultPlanes` or plotMultPlanes first;
  - Input: dictionary of planes and the orientation (vertical or horizontal);
  - Output: single plane bidimensional colormap plot;
- `plotPlane`: plot the measured transverse (vertical) or longitudinal (horizontal) section. This file is a XZ or YZ measurement to compare with the cutPlane plots;
  - Input: complete path of the data file;
  - Output: single plane bidimensional colormap plot.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built with

* [![JupyterNotebook][JupyterNotebook.js]][JupyterNotebook-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## How to use

### File structure

The head of each file has the following information:
- Robot parameters: X, Y, Z, rotation and quternions in the RAPID file;
- Distance AUT/Robot (mm): distance between near-field probe and AUT;
- VNA paramters: start and stop frequencies, number of frequency points ($f_{points}$), IFBW, and frequency generator and multiplier (if used);
- Area: X, Y, and Z edges given in mm and their respective number of points. A 300 mm X edge with 25 points has a step of 12 mm and ranges from -150 to +150 mm.

Data were recorded as Real and Imaginary numbers. Therefore, each file has $n_{cols}$ columns, where

$$
n_{cols} = 4 + 2 * f_{points}
$$

The first column enumerate each measurement. The next 3 columns are the X, Y, and Z position, where the Z position starts at 0 and is incremented by $\frac{Distance (z)}{Points (z)-1}$. The following columns are labeled as the frequency in Hz, where the Real and Imaginary numbers have the same frequency as label. After importing, Python shall attribute a '.1' to the repetead frequency label (the imaginary one).


### Folders and subfolders

Files are separated by frequency band. Each frequency band folder has two subfolders: Planes and Sections.
- Planes folders are XY planes measured at an specifics Z, i.e., an specific distance between probe and antenna under test (AUT);
  - The file contains only the initial distance between probe and AUT (Distance AUT/Robot (mm): 50.0);
  - Other distances are calculated as

$$
Distance AUTRobot + Plane * \frac{Distance (z)}{Points (z)-1}
$$

    As an example, Plane 09 of the X-band lens horn antenna is at:

$$
Distance AUTRobot + Plane * \frac{Distance (z)}{Points (z)-1} = 50 mm + 9 * \frac{300 mm}{20-1} \approx 192,11 mm
$$
    
- Sections are measurements performed in the XZ plane (Longitudinal) or YZ plane (Transverse). Both measurements were performed centered to the propagation wave.

### Code

The following example will demonstrate how to import multiple planes from a folder.

Color maps are plotted based on Pandas DataFrame. Import the data to a DataFrame using
  ```sh
  planesDF = importMultPlanes(pathFile, skipRows=33)
  ```
where pathFile is the path with all data files and skipRows define the number of head lines to skip.

Then, planes can be plotted all at once with

  ```sh
  plotMultPlanes(planesDF, interpolationMethod='hanning', contourLevels=[-10, -3], column=4, mag_phase='Mag Norm Planes', saveFig='no')
  ```

Please notice that `column` must be an even number equals or greater than 4, since the even columns are the real component and the odd ones are the imaginary component.

The standard interpolation method is Hanning, but can choose other method. Please check https://matplotlib.org/stable/gallery/images_contours_and_fields/interpolation_methods.html for other methods.

The standard contour levels are at -10 and -3 dB, but other values can be added. It must be in a list format, in an ascending order.

This function will plot the normalized magnitude over all planes. To plot the magnitude normalized at each plane choose `mag_phase='Mag Norm'`, and to plot the phase choose `mag_phase='Phase'`.

When `saveFig='yes'`, an `.png` image will be exported to the main folder.


A vertical or horizontal cut can be performed over the planes using

  ```sh
  cutPlanes(planesDF, cut=0, orientation='vertical', saveFig='no')
  ```

The `orientation` must be `vertical` or `horizontal`, and `cut=0` means that the cut is centered. Other cutted planes can be plotted, and the value must be a multiple of $\frac{Distance (x)}{Points (x)-1}$ (horizontal) or $\frac{Distance (y)}{Points (y)-1}$ (vertical).

The longitudinal and transverse data files are measurements equivalents to horizontal and vertical cuts, respectively. They can be plotted using

  ```sh
  plotPlane(planesDF, propagation='transverse', column=4, saveFig='no')
  ```

It must be chosen the adequate `propagation`, i.e., `longitudinal` or `transverse`.

This function operates only with a simple normalization, once it is performed a single scan of the plane. Therefore, `mag_phase` can be either `'Mag Norm'`, `'Mag'`, or `'Phase'`.

It is possible to evaluate a single plane and plot it using `plotSinglePlane`. This is demonstrated in the source code with 'Raw' cells.


<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->
## Contact

Alan Boss - alan.boss@inpe.br

Project Link: [https://github.com/afnboss/nf-lens-horn-antenna](https://github.com/afnboss/nf-lens-horn-antenna)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
[JupyterNotebook.js]: https://img.shields.io/badge/Jupyter%20Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white
[JupyterNotebook-url]: https://jupyter.org/
[license-shield]: https://img.shields.io/github/license/afnboss/nf-lens-horn-antenna.svg?style=for-the-badge
[license-url]: https://github.com/afnboss/nf-lens-horn-antenna/blob/master/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/alanboss86
