# JEWELS: JWST Weak-Lensing simple analysis pipeline
A weak-lensing mass mapping pipeline designed for analyzing galaxy cluster images. This pipeline detects and quantifies the deformation effect a galaxy cluster imposes on background light
sources, enabling the inference of the cluster’s projected mass distribution. It was written by using the modules: Numpy, Scipy, emcee, astropy and photutils. The pipeline assumes standard cosmology throughout.

## Documentation:
* _**function**_ **d_A_z** _**(z)**_: returns the distance to a source at redshift z in Mpc.

* _**function**_ **d_A_z1z2** _**(z1, z2)**_: returns the between sources at redshifts z1, z2.

* _**function**_ **Sersic_2D** _**(grid_x, grid_y, params)**_: a galaxy model (Sersic profile, from https://docs.astropy.org/en/stable/api/astropy.modeling.functional_models.Sersic2D.html). This is the model the deconvolver (see below) uses in its operation.
    - **Args:**
<br />**eps** _(int or float; range [0, 1])_: eps = 1 - b/a, where a and b are the semi-major and semi-minor axes of the Sersic ellipse, respectively.
<br />**a** _(int or float; a > 0)_: the semi-major axis of the Sersic ellipse, in pixels.
<br />**phi** _(int or float; range [-pi/2, pi/2])_: the inclination angle of the PIEMD ellipse, in radians.
<br />**sers_ind** _(int or float; range [0.06, 10])_: the Sersic index.
<br />**x0, y0** _(int or float)_: the central coordinates of the Sersic ellipse, in pixels.
    - **returns:**    
Sersic model _(2d ndarray)_

* _**class**_ **PIEMD** _**(grid_x, grid_y, center_x, center_y, eps, phi, sigma_v, r_core, D_l, D_s, D_ls, pixel_in_DD=** 0.04/3600 **, pixels_in_x=** 1 **, pixels_in_y=** 1 **)**_: 
A galaxy cluster parametric model (from "glafic (2022)" appendix B.6 & B.7).
<br /><br />*In the following, FOV = Field Of View, that is, the galaxy cluster image that you try to fit this model to. If you are not using the model for fitting, you can stay with the defaults.

    - **Args:**
<br />**grid_x** _(int or 1d/2d ndarray)_: the x coordiante in pixels to calculate the model on, in pixels.
<br />**grid_y** _(int or 1d/2d ndarray)_: the y coordiante in pixels to calculate the model on, in pixels.
<br />**center_x** _(int or float)_: the x coordinate of the PIEMD ellipse center, in pixels.
<br />**center_y** _(int or float)_: the y coordinate of the PIEMD ellipse center, in pixels.
<br />**eps** _(int or float; range [0, 1])_: eps = 1 - b/a, where a and b are the semi-major and semi-minor axes of the PIEMD ellipse, respectively.
<br />**phi** _(int or float; range [-pi/2, pi/2])_: the inclination angle of the PIEMD ellipse, in radians.
<br />**sigma_v** _(float; sigma_v > 0)_: the velocity dispersion of the galaxies in the modeled galaxy cluster, in km/sec.
<br />**r_core** _(float; r_core > 0)_: the core radius of the PIEMD ellipse, in Mpc.
<br />**D_l** _(float; D_l > 0)_: the distance from the observer to the lens, in Mpc.
<br />**D_s** _(float; D_s > 0)_: the distance from the observer to the source, in Mpc.
<br />**D_ls** _(float; D_ls > 0)_: the distance from the lens to the source, in Mpc.
<br />**pixel_in_DD** _(int or float; pixel_in_DD > 0)_: the FOV resolution, i.e., how many decimal degrees does one pixel represent. The defualt is 0.04/3600, 
                                            that is, 1 pixel = 0.04".
<br />**pixel_in_x** _(int or float; pixel_in_x > 0)_: for cases the grid you use has a lower resolution than the FOV. It sets the number of FOV pixels that fit 
                                          into a grid pixel, in the x direction. The default is 1 (i.e., the same resolution).
<br />**pixel_in_y** _(int or float; pixel_in_y > 0)_: for cases the grid you use has a lower resolution than the FOV. It sets the number of FOV pixels that fit 
                                          into a grid pixel, in the y direction. The default is 1 (i.e., the same resolution).

* _**method**_ **.PIEMD_second_derivatives()**: returns the second derivatives psi_xx, psi_yy, psi_xy. Their shape is the same as the shape of grid_x and grid_y. But subtracting 2 different **.PIEMD_second_derivatives()**, one gets a _dPIE model_.

* _**method**_ **.kappa()**: returns the convergence: 0.5 * (psi_xx + psi_yy)  

* _**method**_ **.gamma_1()**: returns the shear's 1st component: 0.5 * (psi_xx - psi_yy)  

* _**method**_ **.gamma_2()**: returns the shear's 2nd component: psi_xy 

* _**method**_ **.gamma()**: returns the shear's magnitude: sqrt(gamma_1^2 + gamma_2^2)  

* _**method**_ **.angle()**: returns the shear's inclination angle: 0.5 * arctan2(gamma_2, gamma_1)


* _**class**_ **light_sources** _**(x, y, a, b, phi, cluster_image, seg_map)**_: A class for the catalog of a galaxy cluster.
    - **Args:**
<br />**x** _(1d ndarray)_: the x coordinates of the catalog galaxies, in pixels.
<br />**y** _(1d ndarray)_: the y coordinates of the catalog galaxies, in pixels.
<br />**a** _(1d ndarray)_: the semi-major axis of the catalog galaxies, in pixels.
<br />**b** _(1d ndarray)_: the semi-minor axis of the catalog galaxies, in pixels.
<br />**phi** _(1d ndarray)_: the inclination angle of the catalog galaxies, in radians.
<br />*All the above arrays must have the same length.
<br />**cluster_image** _(2d ndarray)_: the FOV image.
<br />**seg_map** _(2d ndarray)_: the segmantation map.
<br />*cluster_image.shape == seg_map.shape

* _**method**_ **.size_filter** _**(size=** 20 **)**_: A method that filters the galaxies according to np.size > size.
    - **Args:**
<br />**size** _(int, size > 0)_: the array size bar below which galaxies are filtered. Default is 20.
    - **returns:**
<br />**ind_size** _(1d ndarray)_: the indexes of the galaxies with np.size > size.

* _**method**_ **.deconvolver** _**(ID_ind, PSF, nsteps=** 10000 **, MCMC_progress=** False **)**_: A method that deconvolves each galaxy image from the aperture's PSF, using MCMC.
Can be parallelized (see example notebook). It assumes a Sersic profile for the intrinsic galaxy, and accounts for the model bias.
    - **Args:**
<br />**ID_ind** _(int)_: the galaxy's index to be deconvolved.
<br />**PSF** _(2d ndarray)_: the aperture's PSF. Must be normlized such that np.allclose(np.sum(PSF), 1, 1e-1, 1e-1).
<br />**nsteps** _(int)_: the number of steps for the MCMC walkers to take. Default is 10000.
<br />**MCMC_progress** _(bool)_: whether to show the progress bar or not. Default is False.
<br /><br />*Below, ellipticity is e = (a - b)/(a + b) that is need for WL; not to be confused with eps = 1 - b/a.

    - **returns:**
<br />**ID_ind** _(float)_: the deconvolved galaxy's index.
<br />**e1** _(float)_: the deconvolved galaxy's first ellipticity component, that is, e = cos(2phi), where phi is the inclination angle.
<br />**e2** _(float)_: the deconvolved galaxy's second ellipticity component, that is, e = sin(2phi), where phi is the inclination angle.
<br />**ind_gal** _(float)_: the deconvolved galaxy's segmantation map index (as it might not be the same as ID_ind).

* _**method**_ **.shear** _**(e1, e2, x, y, divx, divy, SN)**_: A method that calculates the shear over a grid overlaying the galaxy cluster's FOV.
    - **Args:**
<br />**e1** _(1d ndarray)_: 1st ellipticity component of all galaxies to be considered (see 'deconvolver').
<br />**e2** _(1d ndarray)_: 2st ellipticity component of all galaxies to be considered (see 'deconvolver').
<br />**x** _(1d ndarray)_: x coordinate of all galaxies to be considered.
<br />**y** _(1d ndarray)_: y coordinate of all galaxies to be considered.
<br />**divx** _(int, divx > 0)_: the number of x axis divisions.
<br />**divy** _(int, divy > 0)_: the number of y axis divisions.
<br />**SN** _(1d ndarray)_: the flux S/N values of all galaxies to be considered.
<br /><br />*divx and divy set the grid over which the averaging and calculation of the shear takes place. Only cells with at least 8 sources are considered, in order to get reliable averages.

    - **returns:**
<br />**g1** _(2d ndarray)_: 1st shear component grid.
<br />**g1_std** _(2d ndarray)_: 1st shear component std grid.
<br />**g2** _(2d ndarray)_: 2st shear component grid.
<br />**g2_std** _(2d ndarray)_: 2st shear component std grid.
<br />**g** _(2d ndarray)_: shear magnitude grid.
<br />**phi_mean** _(2d ndarray)_: shear inclination angle grid.
<br />**jacobian_xy** _(float)_: the jacobian transforming from the FOV resolution to the shear grid resolution.
<br />**x_axis_cells** _(1d ndarray)_: shear grid's x axis.
<br />**y_axis_cells** _(1d ndarray)_: shear grid's y axis.
<br /><br />*shear is defined similarly to ellipticity (see 'deconvolver').

* _**method**_ **.convergence_map** _**(axis_1, axis_2, shear1, shear2, jacobian, divx, divy)**_: A method that computes the convergence over the same grid that the shear was calculate on.
    - **Args**:
<br />**axis_1** _(1d ndarray)_: shear grid's x axis.
<br />**axis_2** _(1d ndarray)_: shear grid's y axis.
<br />**shear1** _(2d ndarray)_: 1st shear component grid (see 'shear').
<br />**shear2** _(2d ndarray)_: 2st shear component grid (see 'shear').
<br />**jacobian** _(float)_: the jacobian transforming from the FOV resolution to the shear grid resolution.
<br />**divx** _(int, divx > 0)_: the number of x axis divisions.
<br />**divy** _(int, divy > 0)_: the number of y axis divisions.
    - **returns:**
<br />**kappa** _(2d ndarray)_: the convergence grid.

* _**attribute**_ **.convergence_uncertainty**: The uncertainty of the convergence map.

* _**function**_ **mass** _**(convergence, mask, D_l, D_s, D_ls, x_min, x_max, y_min, y_max, divx, divy, pixel_in_DD=** 0.04/3600 **)**_: A function that estimates the mass of the galaxy cluster, based on its convergence map
    - **Args:**
<br />**convergence** _(2d ndarray)_: the convergence map.
<br />**mask** _(tuple of ndarrays, ndarray or int)_: the array elements you wish to sum over.
<br />**D_l** _(float; D_l > 0)_: the distance from the observer to the lens, in Mpc.
<br />**D_s** _(float; D_s > 0)_: the distance from the observer to the source, in Mpc.
<br />**D_ls** _(float; D_ls > 0)_: the distance from the lens to the source, in Mpc.
<br />**x_min** _(int)_: the minimal value of the x axis of the convergence map.
<br />**x_max** _(int)_: the maximal value of the x axis of the convergence map.
<br />**y_min** _(int)_: the minimal value of the y axis of the convergence map.
<br />**y_max** _(int)_: the maximal value of the y axis of the convergence map.
<br />**divx** _(int, divx > 0)_: the number of x axis divisions.
<br />**divy** _(int, divy > 0)_: the number of y axis divisions.
<br />**pixel_in_DD** _(int or float; pixel_in_DD > 0)_: the FOV resolution, i.e., how many decimal degrees does one pixel represent. The defualt is 0.04/3600, 
                                             that is, 1 pixel = 0.04".    
    - **returns:**
<br />**mass** _(float)_: the mass estimate of the cluster, in solar masses.
<br />**mass_uns** _(float)_: the uncertainty of the above value, taken to be 15% of it, following Zitrin et al. (2015)*.
<br /><br />*Adi Zitrin _et al_ 2015 _ApJ_ **801** 44. **DOI** 10.1088/0004-637X/801/1/44.
