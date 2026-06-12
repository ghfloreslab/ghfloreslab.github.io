---
layout: page
title: Scientific Visualization
description: Computer Graphics
img: assets/img/project2/project2thumbnail.png
importance: 2
category: academic
---

<h2>Introduction</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">

Scientific Visualization is the visualization of 3-Dimensional phenomena (medical, biological, metereological, etc..) with the purpose to render meaningful volumes and surfaces. An area of scientific visualization that is very common and useful is medical imaging. The data collected from a tomographic system such as a CT or X-ray system consists of a set of parallel planes that can be viewed as a 3D volume once all the planes are put together in a pre-defined sequence. Each of these planes contains a particular property such as density, color or X-ray absorption coefficient that was captured at an instant in time.

In the past few years, advancements in hardware and software has made it feasible to render large volumetric datasets acquired by these scanning devices since these datasets can range from few megabytes to a several gigabytes in size depending on the resolution. An example is the <a href="https://www.nlm.nih.gov/databases/download/vhp.html">Visible Human Project</a> which consists of MRI, X-ray CT and anatomical images obtained from cadavers. Each of the datasets, male and female cadavers, contains 15 gigabytes and 40 gigabytes of voxel data sets, respectively. In order to visualize these large volumetric datasets, each property value can be represented as an individual volumetric unit, or voxel. Each voxel encapsulates several properties such as (x, y, z) location, density, color, gradient, normal, etc.

Several techniques exist that allow for the extraction of meaningful information from these volumetric datasets. The meaningful information must only contain relevant data while ignoring all other data. For example, a CT scan of a baby head must only show the head while trying to avoid the respiratory tube and head padding as well as other artifacts. This is very important in order to allow the clinician to efficiently and precisely diagnose and screen for a disease.

This project has been designed and implemented to model the "Bubble model" algorithm described in <a href="https://www.researchgate.net/profile/Eduard-Groeller/publication/221474886_Interactive_Volume_Rendering_based_on_a_Bubble_Model/links/0912f5113d9408f68c000000/Interactive-Volume-Rendering-based-on-a-Bubble-Model.pdf">[1]</a>, which allows for understandable data representation, quick data manipulation, and reasonable rendering times. Understandable data representation is accomplished by implementing the "Bubble model" algorithm and then combining it with Ray casting to do surface shading. A graphical user interface using <a href="https://github.com/libglui/glui">GLUI</a>, a C++ GUI library, is created to allow the user to change various parameters such as opacity, threshold values, and rendering techniques.

<h2>Goals | Problem Statement</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">

<div class="card" style="width: auto;">
  <div class="card-header" style="text-align: center;">
    Learn about techniques commonly used to visualize CT or MRI data and to be exposed to the concepts of volume rendering. Understanding these techniques and implementing some of them will prove useful in learning the pros and cons of these techniques as well as the ideas and terminology currently being used in the medical field of scientific visualization.
  </div>
</div>
<br>
<h2>Background</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
Volume visualization has been an active area of research during the past 20 years. In order to visualize the data efficiently and precisely, volume visualization techniques must offer the following characteristics:
<ol>
    <li>Understandable data representation</li>
    <li>Quick data manipulation</li>
    <li>Reasonable rendering times</li>
</ol>
There are four classes of classical volume rendering algorithms <a href="https://dl.acm.org/doi/pdf/10.1145/192161.192283">[2]</a>:
<div class="card">
    <div class="card-body">
        <h3 class="card-title"></h3>
        <ol>
            <li>
            <strong>Ray Casting:</strong> a ray is casted through the volume of each image pixel and the color and opacity values along the ray are integrated
            </li>
            <li>
            <strong>Splatting:</strong> contribution of a voxel to the image is computed by distributing the voxel's value to a neighborhood of pixels
            </li>
            <li> 
            <strong>Cell Projection:</strong> the volume is decomposed into polyhedra whose vertices are the sample points. Then the polyhedra is sorted by depth order and the polygonal faces are scan converted into image space. The color and opacity at each pixel are computed between the front and back faces of each polyhedron
            </li>
            <li>
            <strong>Multi-pass resampling:</strong> the entire volume is resampled to the image coordinate system so that the resampled voxels line up behind each other on the viewing axis in image space. Then the voxels are composited together along the viewing axis as in a ray caster
            </li>
        </ol>
    </div>
</div>
<br>
This family of algorithms are robust and good at representing the data in an understandable manner. Yet, the large 3D arrays of data must be sampled and therefore are computationally expensive. As shown in figure 1, a dataset of a human head could contain 15 million samples. Iterating through all these samples in a 3-Dimensional dataset is computationally costly and is typically $$O(n^3)$$.
<div class="text-center">
        {% include figure.html path="assets/img/project2/ctscan_humanhead.png" title="CT scan human head image" class="img-fluid rounded z-depth-1" width="auto" height="auto" %}    
        <div class="caption">
            Figure 1. CT scan of a human head. The data input contains 15 million samples. [2]
        </div>
</div>
Given enough rendering time, processing power and high memory bandwidth, these datasets can be easily rendered. Yet, two of the characteristics, quick data manipulation and reasonable rendering times, cannot be achieved and does not allow for interactive applications. In order to cover the latter characteristics, algorithmic optimizations that exploit coherence in the data can be implemented, thus achieving high rendering rates and allowing for interactive applications without the need of special hardware.
<br>
<h2>Design</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
This project was designed to handle 8-bit and 16-bit .raw format datasets. These datasets were acquired from the <a href="http://www.volvis.org"><s>Volume Dataset Repository</s> (no longer available)<a> at the WSI/GRIS, University of Tübingen, Germany, and the <a href="http://celebisoftware.com">Celebi Software public repository of medical imaging</a>. While some datasets were downloaded in .raw format, many others were in .pvm format. Data stored in .pvm format contains information about bit depth, cell spacing, grid size, dataset description, type of scanner, comments and raw data. The .raw format was chosen for this project. In order to convert from .pvm to .raw format, the utility provided by the v^3 Volume rendering Package (PVM tools) provided by the Volumne Dataset Repository. Once the .raw format file was generated by either using the v^3 utility or downloading it from a database, the file was then read by my application and processed as shown in Figure 3.
<div class="text-center">
        {% include figure.html path="assets/img/project2/volumeprocessingflowchart.png" title="volume processing flow chart image" class="img-fluid rounded z-depth-1" width="auto" height="auto" %}    
        <div class="caption">
            Figure 3. Volume processing flow chart.
        </div>
</div>
When the file is read by the application, it is processed depending on the bit data type:
<ol>
    <li>
    If the data is in 8-bit format, the gradient thresholds are calculated, voxels are generated and saved in a vector container if the value meets the threshold level. This data is then displayed according to the user's selected rendering option: Render as voxels, Bubble Model or ray casting.
    </li>
    <li>
    If the data is in 16-bit format, the gradient tresholds are calculated and a simple version of a voxel is generated and saved in a vector container if the value meets a given treshold level. The data is then displayed according to the user's rendering option.
    </li>
</ol>
A simple version of a voxel is used for 16-bit data since these datasets tend to be of size (512x512x512) rather than (256x256x256) as seen in must 8 bit datasets. Due to the big size difference, processing of large datasets tends to consume a lot of memory and the application is not able to allocate the memory required if the Voxel class contains many data members. Therefore, after several hours of testing and figuring out that this was happening to my application, a Voxel structure was created and used rather than a Voxel class. In addition, rather than creating a voxel that contained all 8 vertices, only 1 vertex was saved and all other Voxel class functions omitted in order to save space and be able to render 16-bit datasets. Overall, a total of four classes were implemented:
<ol class="list-group list-group-numbered">
  <li class="list-group-item"><strong>Vertex class:</strong> Class that saves the (x,y,z) coordinate values and the normal values</li>
  <li class="list-group-item"><strong>Vector3D class:</strong>
          Class that saves the (x, y, z) coordinate values, the normal values, and the calculated vector magnitude. It also includes overloaded
           multiplication, adddition, and subtraction operators as well as the dot product of two vectors</li>
  <li class="list-group-item"><strong>Voxel8bit class:</strong>
          Class that saves all eight vertices of a voxel, the gradient magnitude, color, opacity, and normal. Set and get methods were
           provided to change these values</li>
  <li class="list-group-item"><strong>Voxel16bit class:</strong>
          Class that saves all eight vertices of a voxel, the gradient magnitude, color, opacity, and normal. Set and get methods were
           provided to change these values. This class is not used due to memory limitations.</li>
  <li class="list-group-item"><strong>main.cpp Driver:</strong>
          This driver contains all the code to create the graphical user interface and all algorithms to process 8-bit and 16-bit volumetric datasets. It also displays these datasets according to the rendering option selected: Voxels, Bubble Model, and Ray Casting</li>
</ol>
<br>
Using the classes mentioned above, the data saved and stored in the vector container was extracted and displayed according to the Algorithms section, and a GUI using GLUI library was created as shown in figure 4.
<div class="text-center">
        {% include figure.html path="assets/img/project2/graphicaluserinterfaceexample.png" title="graphical user interface example image" class="img-fluid rounded z-depth-1" width="auto" height="auto" %}    
        <div class="caption">
            Figure 4. Graphical user interface.
        </div>
</div>

The following options are allowed to be changed by the user:
<ol>
    <li>Zoom: zoom in and out to make the volume rendered bigger or smaller</li>
    <li>Opacity: Increase or decrease the opacity value to make the surfaces darker or lighter</li>
    <li>Select the type of rendering: Voxel, Bubble model, or ray casting</li>
    <li>Animate the volume by having it rotate along the y axis</li>
    <li>Show (x, y, z) axes</li>
    <li>Display the volume as greyscale or color</li>
    <li>Change the threshold value to add or remove iso-surfaces. The min and max threshold values are displayed</li>
</ol>

The dataset resolution, dataset name, minimum and maximum allowed threshold values calculated by gradient magnitudes are shown in the upper right corner of the application using Bitmap characters.
<br>
<h2>Algorithm</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
In medical imaging it is important to reduce the rendering time in order to allow for user interaction. For example, a radiologist or lab technician must be able to diagnose a critical injury right after it happened so that the patient can be treated accordingly and efficiently. In order to avoid the large rendering time, the <a href="https://www.researchgate.net/profile/Eduard-Groeller/publication/221474886_Interactive_Volume_Rendering_based_on_a_Bubble_Model/links/0912f5113d9408f68c000000/Interactive-Volume-Rendering-based-on-a-Bubble-Model.pdf">"Bubble Model"</a> technique was implemented to ensure fast previewing of volumetric data. The algorithm described in this paper, and summarized below, works as follows:
<div class="card">
    <div class="card-body">
        <h3 class="card-title">Bubble Model</h3>
        The main idea of the Bubble model algorithm is to render multiple iso-surfaces as thin semi-transparent membranes. Decreasing the opacity assigned to a given iso-surface, causes anything behind the iso-surface to be more visible. The model works as follows:
        <br><br>
        <ol>
            <li>
                The surfaceness of a voxel is characterized by the gradient magnitude at the voxel position. If the gradient is within a threshold value, then the voxel belongs to an isosurface. Otherwise, the voxel belong to a homogeneous region and it is not taken into account. The gradient is calculated by central differences as follows:
                $$\nabla f(x_i, y_j, z_k)\thickapprox
                    \dfrac{1}{2}\left[
                        \begin{array}{c}
                        f(x_{i+1}, y_j, z_k) - f(x_{i-1}, y_j, z_k)\\
                        f(x_{i}, y_{j+1}, z_k) - f(x_{i}, y_{j-1}, z_k)\\               
                        f(x_{i}, y_{j}, z_{k+1}) - f(x_{i}, y_{j}, z_{k-1})\\ 
                        \end{array}
                    \right]
                $$
            </li>
            <li>
                Opacities proportional to the gradient magnitudes are calculated and assigned to the voxels. The following equation shows how opacity is calculated. $s$ in this equation is a scalar constant factor that can be tweaked by the user to increase or decrease transparency and improve visualization.
                $$\alpha_{i, j, k} = \lvert \nabla f(x_i, y_j, z_k) \rvert s$$
            </li>
            <li>
                A constant ambient background light is assumed. Therefore, each voxel within a treshold level attenuates the background light. Each pixel intensity is calculated as an accumulated transparency multiplied by the ambient light.
            </li>
        </ol>
        The advantages of this algorithm are (1) due to the opacity function weighted by gradient magnitudes, the total number of voxels contributing to the final image is reduced, and (2) the user interaction is reduced thus allowing for tuning of rendering parameters and immediate feedback.
    </div>
</div>
<br>
<div class="card">
    <div class="card-body">
        <h3 class="card-title">Ray Casting</h3>
        The Bubble Model can be combined with traditional surface shaded display. The viewing rays are evaluated as follows:
        <br><br>
        <ol>
            <li>
            The volume is resampled along a viewing ray defined by an origin and direction vectors
            </li>
            <li>
            The density value and gradient magnitude at each sample point are calculated using trilinear interpolation
            </li>
            <li> 
            If density is greater than a specified treshold value, the shaded color of the hit multiplied by the accumulated transparency is returned
            </li>
            <li>
            Otherwise, the opacity is multiplied by a scaling factor and contributes to the accumulated trasnparency
            </li>
        </ol>
        These 3 steps are coded as follows (reproduced from <a href="https://www.researchgate.net/profile/Eduard-Groeller/publication/221474886_Interactive_Volume_Rendering_based_on_a_Bubble_Model/links/0912f5113d9408f68c000000/Interactive-Volume-Rendering-based-on-a-Bubble-Model.pdf">[1]</a>):
{% highlight c++ linenos %}
double rayCasting(Volume volume, Vec3D origin, Vec3D direction)
{
    double transparency = 1.0;

    for(int i=0; i<i_max; i++)
    {
        Vec3D sample = origin + direction * i;
        Voxel voxel = volume.Resample(sample);

        if(voxel.density > threshold) 
        {
            return Shading(voxel) * trasnparency;
        } else {
            double opacity = voxel.gradient_magnitude * scaling_factor;
            transparency *= 1.0 - opacity;
        }
    }
    return ambient_light * transparency;
}
{% endhighlight %}
    </div>
</div>
<br>
<br>
<h2>Results</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
The application was implemented in C++ using Xcode on an Intel Core 2 Duo 2.33 Ghz with 2 GB of RAM. Three of the four goals are functioning properly. The Ray Casting portion of the code has been implemented but is not working as expected. The time consuming part of the volume rendering is the preprocessing time needed to calculate the min and max threshold values and generate the volume within that threshold value. Table 1 shows various preprocessing times for 4 datasets, 2 8-bit and 2 16-bit datasets of various resolution sizes. As shown in table 1, the 8 bit datasets typically take 40 seconds to do the preprocessing step, while the 16-bit dataset of resolution (512x512x512) takes about 9 minutes and 30 seconds.
<div class="text-center">
        {% include figure.html path="assets/img/project2/processingtimestable.png" title="processing times table image" class="img-fluid rounded z-depth-1" width="auto" height="auto" %}    
        <div class="caption">
            Table 1. Preprocessing times for 4 datasets, 16-bit and 8-bit of different resolution sizes.
        </div>
</div>

Before 16-bit datasets could be rendered, several modifications were made to render the volume. Initially, a Voxel16bit class was implemented and contained several member types: 8 float vertices, a Vector3D gradient, a float gradient magnitude, a Vector3D normal plus all set, get, constructors and destructor methods. Saving voxels of this type in a vector container generated memory allocation errors. Therefore, instead of creating a Voxel16bit class, a Voxel16 structure with the minimum number of members was used to decrease the amount of memory needed to store all the voxels required to render a 16-bit dataset of resolution (512x512x512).

<div class="accordion" id="accordionPanelsStayOpenResults">
    <div class="accordion-item">
        <h2 class="accordion-header">
            <button class="accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseVoxel" aria-expanded="true" aria-controls="panelsStayOpen-collapseVoxel">
            Rendered Voxels Only
            </button>
        </h2>
        <div id="panelsStayOpen-collapseVoxel" class="accordion-collapse collapse show">
            <div class="accordion-body">
                8-bit dataset with resolution (256x256x98) and a 16-bit dataset with resolution (512x512x512) results. The Render Voxels option renders the voxels created from the data extracted for a given threshold value.
                <br><br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_1_1.png" title="results 1-1 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_1_2.png" title="results 1-2 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                </div>
                <div class="caption">
                    8-bit dataset of a baby head, and 16-bit dataset of the vertebra.
                </div>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseBubbleModel" aria-expanded="false" aria-controls="panelsStayOpen-collapseBubbleModel">
                Rendered using Bubble Model
            </button>
        </h2>
        <div id="panelsStayOpen-collapseBubbleModel" class="accordion-collapse collapse">
            <div class="accordion-body">
                8-bit and 16-bit datasets are rendered using the Bubble Model. The opacity of the voxels can be increased or decreased. An understandable representation of the volume is rendered and interactive.
                <br><br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_2_1.png" title="results 2-1 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_2_2.png" title="results 2-2 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                </div>
                <div class="caption">
                    8-bit dataset of a baby head and 16-bit dataset of the vertebra using the Bubble Model.
                </div>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <h2 class="accordion-header">
            <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#panelsStayOpen-collapseRayCasting" aria-expanded="false" aria-controls="panelsStayOpen-collapseRayCasting">
            Ray Casting
            </button>
        </h2>
        <div id="panelsStayOpen-collapseRayCasting" class="accordion-collapse collapse">
            <div class="accordion-body">
                8-bit and 16-bit datasets are rendered using the Ray Casting method to provide surface shading. Only one slice of the baby head dataset was rendered showing the shading, and the Bucky Ball dataset was rendered showing the shaded corners and edges.
                <br><br>
                <div class="row justify-content-sm-center">
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_3_1.png" title="results 3-1 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                    <div class="col-sm-6 mt-3 mt-md-0">
                        {% include figure.html path="assets/img/project2/results_3_2.png" title="results 3-2 image" class="img-fluid rounded z-depth-1" %}
                    </div>
                </div>
                <div class="caption">
                    (256x256) slice of a  baby head and a bucky ball using Ray casting algorithm.
                </div>
            </div>
        </div>
    </div>
</div>
<br>
<h2>Conclusion</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
In conclusion, the "Bubble Model" produces interactive renderings with fairly enough detail to distinguish between different surfaces in a volumetric dataset. Due to the gradient being calculated by central differences, staircase artifacts are seen during the rendering. More sophisticated gradient methods exist but increases the preprocessing time [3]. This was exciting and fun project since I implemented different algorithms and techniques that exist and that provide extremely good amount of detail. The "Bubble model" provides a good visualization of a volume. However, in order to get a lot of detailed information, another rendering technique should be used. To understand the scientific datasets and to make sure I was reading the data correctly, I render the voxels of those data elements that meet a certain threshold level. Then I render the scientific volume using the "Bubble model" algorithm. Interaction is possible and allows the user to change various parameters at run time. The Ray Casting technique was implemented but it does not work as expected. Only a slice of a large dataset was rendered to see the shaded surface. For a small dataset, the external surface is shaded but the entire volume is rendered. I believe the error is during the trilinear interpolation section and volume sampling of the algorithm.
<br>
<h2>References</h2>
<hr class="bg-danger border-2 border-top border-primary-subtle">
[1] Csebfalvi, B and Groller, E. "Interactive Volume Rendering based on a Bubble Model".
         Institute of Computer Graphics and Algorithms,Vienna University of Technology,
         pp. 1-8.
<br><br>
[2]  Lacroute, G. P. "Fast volume rendering using a shear-warp factorization of the viewing 
         transformation". Technical Report. Computer Systems Laboratory, Stanford University,
         pp. 1-236, September 1995.
<br><br>
[3] Neumann, L., Csebfalvi, B., Konig, A., and Groller, E. "Gradient Estimation in Volume
         Data using 4D Linear Regression". Eurographics 2000, Vol 19, No. 3.
<br><br>
[4] Levoy, M. "Display of Surfaces from Volume Data". IEEE Computer Graphics and
         Applications, 8(5):29-37, May 1988.