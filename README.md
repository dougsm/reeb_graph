Description
---------------------

Topological similarity estimation for 3D models (face-vertex meshes) using multiresolutional Reeb graphs (MRG).

The MRG method was proposed by Hilaga et al. in [[1]](#references). 
This implementation was used to obtain experimental results that were reported in [[2]](#references) and [[3]](#references). 

### Implementation Notes

 - Programming language: **Java**
 - Software Prerequisites: JavaSE 1.4+
   - the code was developed using JavaSE 1.4, so it does not use Java generics (introduced in JavaSE 5.0)
 - **3D models must be specified in specially-formatted VRML files** (see [below](#cad-models))
   - Face-vertex meshes with triangle and quad faces are supported
   - 16 sample models are included in this package

Package Contents
---------------------

### Source Code

The source code is located in [`src/`](src/) directory. There are **two** main java classes:  

1. [`ExtractReebGraph`](src/ExtractReebGraph.java) class constructs MRGs for 3D models and saves them into text files.
  - This procedure for MRG construction is described in Section 4 of [[1]](#references).
  - **Usage:**  
    `java ExtractReebGraph`  `<num_pts>`  `<mu_coeff>`  `<mrg_size>`  `<model_1>.wrl`  `<model_2>.wrl` ... `<model_N>.wrl`  
    **where:**
      + `<num_pts>`   &ndash; target number of vertices prior to MRG construction (triangle faces are resampled to match `<num_pts>`)
      + `<mu_coeff>`    &ndash; coefficient for calculating threshold parameter `r=sqrt(mu_coeff * area(S))`, which in turn is used to approximate values of function `mu` in [[1]](#references)
      + `<mrg_size>`   &ndash; number of ranges in the finest resolution of MRG (parameter `K` in [[1]](#references))
      + `<model_i>.wrl`   &ndash; i-th VRML model for `i=[1,N]` (MRG for each model is stored in `<model_i>.mrg`)
2. [`CompareReebGraph`](src/CompareReebGraph.java) class implements matching algorithm for a pairwise comparison of MRGs.
  - The matching algorithm is described in Section 5 of [[1]](#references).
  - **Usage:**  
    `java CompareReebGraph`  `<num_pts>`  `<mu_coeff>`  `<mrg_size>`  `<sim_weight>`  `<model_1>.wrl` ... `<model_N>.wrl`  
    **where:**
      + `<num_pts>`   &ndash; target number of vertices prior to MRG construction (triangle faces are resampled to match `<num_pts>`)
      + `<mu_coeff>`    &ndash; coefficient for calculating threshold parameter `r=sqrt(mu_coeff * area(S))`, which in turn is used to approximate values of function `mu` in [[1]](#references)
      + `<mrg_size>`   &ndash; number of ranges in the finest resolution of MRG (parameter `K` in [[1]](#references))
      + `<sim_weight>`   &ndash; weight `w` used in similarity function (trade-off between attributes `a` and `l` in [[1]](#references))
      + `<model_i>.wrl`   &ndash; a list of VRML models to compare for `i=[1,N]` (it is assumed that each  model was processed using `ExtractReebGraph` program, and MRG for `<model_i>.wrl` was stored in `<model_i>.mrg`)

### CAD Models 

The following 3D models in VRML format can be found in [`models/`](models/) directory:

<a target="_blank" href="https://raw.github.com/dbespalov/reeb_graph/master/figs/sample_models.pdf"><img  width="300px"  src="https://raw.github.com/dbespalov/reeb_graph/master/figs/sample_models.png"/></a>


VRML parser in [`ExtractReebGraph`](src/ExtractReebGraph.java) can **only** handle specially-formatted face-vertex meshes:

* Vertex lists are assumed to be enclosed by strings `point [\n` and `]\n`
  * Coordinates for each point must appear on a separate line in the VRML file (e.g., `1.5 3.2 0.2,\n`)
* Face lists are assumed to be enclosed by strings `coordIndex [\n` and `]\n`
  * Each face must appear on a separate line (e.g., `0, 2, 1, -1,\n` encodes a triangle face)
  

Sample Usage
---------------------

### Compiling

```bash
$ javac src/*.java
```

### MRG Construction

3D models must be saved in a specially-formatted VRML files (see [CAD Models](#cad-models))

```bash
$ ls -1 models/*.wrl  | head -5

models/bracket_1.wrl
models/bracket_2.wrl
models/bracket_3.wrl
models/fork_1.wrl
models/fork_2.wrl
```

Compute MRGs for all `*.wrl` files in a directory:

```bash
$ ls models/*.wrl  | xargs  java -cp "./src/" -Xmx1024m ExtractReebGraph   4000 0.0005 128
```

MRGs are saved into `*.mrg` files:

```bash
$ ls -1 models/*.mrg  | head -5

models/bracket_1.mrg
models/bracket_2.mrg
models/bracket_3.mrg
models/fork_1.mrg
models/fork_2.mrg
```

### MRG Matching

Compute pairwise similarity values for 3D models using their MRG representation:

```bash
$ ls models/*.wrl  | xargs  java -cp "./src/" -Xmx1024m CompareReebGraph   4000 0.0005 128 0.5
```

The similarity values are stored in `log_<pts_num>_<mu_coef>_<mrg_size>_<sim_weight>` file:

```bash
$ shuf log_4000_5.0E-4_128_0.5 | head -5

Similarity between models/goodpart_2.wrl and models/fork_3.wrl is 0.7661396393161327
Similarity between models/housing_1.wrl and models/socket_2.wrl is 0.6789623740585898
Similarity between models/fork_3.wrl and models/bracket_2.wrl is 0.8125245864576977
Similarity between models/goodpart_1.wrl and models/socket_2.wrl is 0.8162694461373452
Similarity between models/bracket_3.wrl and models/housing_1.wrl is 0.6865232310951028
```

### Retrieval Results

Pairwise similarity values can be used to rank 3D models in terms of their relevance to a query model. Five top-ranked models for selected queries are:

<a target="_blank" href="https://raw.github.com/dbespalov/reeb_graph/master/figs/sample_matches.pdf"><img  width="400px"  src="https://raw.github.com/dbespalov/reeb_graph/master/figs/sample_matches.png"/></a>


License
---------------------
GNU General Public License


References
---------------------

1. Masaki Hilaga, Yoshihisa Shinagawa, Taku Kohmura, and Tosiyasu L. Kunii.  
   Topology Matching for Fully Automatic Similarity Estimation of 3D Shapes.  
   *SIGGRAPH*, 2001. [doi:10.1145/383259.383282](http://dx.doi.org/10.1145/383259.383282) 

2. Dmitriy Bespalov, William C. Regli, and Ali Shokoufandeh.  
   Reeb graph based shape retrieval for CAD.  
   *ASME IDETC*, 2003. [doi:10.1115/DETC2003/CIE-48194](http://dx.doi.org/10.1115/DETC2003/CIE-48194)

3. Dmitriy Bespalov, Cheuk Yiu Ip, William C. Regli, and Joshua Shaffer.  
   Benchmarking search techniques for CAD.  
   *ACM SPM*, 2005. [doi:10.1145/1060244.1060275](http://dx.doi.org/10.1145/1060244.1060275)
