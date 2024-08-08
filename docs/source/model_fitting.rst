.. _model_fitting:

Model Fitting
=============

While fixed models (i.e. models that predict a fixed RDM) are important, many models have free parameters that can be fit to the data. All models come equipped with a default fitting function,
which is called using the ``fit`` function of the model. This function takes a data RDMs object as input and returns a parameter value.

Individual vs. group fits
-------------------------
In the RSA toolbox, the models will always be fit to all RDMs that are passed to the fitting routine. That is, if the different RDMs are calculated on different subjects,
the fit will be a group fit, with one set of parameters for the whole group. If you require individual fits, you need to loop over participants pass each RDM seperately to the fitting function.

Continuous vs. rank-based evaluation metrics
--------------------------------------------
Most model fitting techniques require the evaluation metric to be differentiable with regard to the model parameters and this is true for most
RDM comparison metrics provided by the toolbox. However, rank based evaluations (i.e. Spearman $\rho$ and Kendall's $\tau$) are not differentiable
and thus often lead to problems for optimization. Thus, we generally do not recommend fitting models using these criteria.

Two model types with their special fitting methods are exempt from this recommendation: :ref:`Selection models <ModelSelect>` and
:ref:`Interpolation models <ModelInterpolate>`, which use a simple selection strategy and a derivative free bisection method for finding
the best models respectively and can thus deal with discontinuous objective functions.

Fitting algorithms
------------------

Different fitting methods can be found in ``rsatoolbox.model.fitter`` module.
To use them you can either apply them manually you can set them as the default of a model by settings its ``default_fitter`` property.

Unconstrained optimization
^^^^^^^^^^^^^^^^^^^^^^^^^^
The function ``rsatoolbox.fitter.fit_optimize`` uses a general purpose optimization method. It simply maximizes the criterion using either positive or negative ``theta`` weights.

Non-negative optimization
^^^^^^^^^^^^^^^^^^^^^^^^^
The function ``rsatoolbox.fitter.fit_optimize_positive`` maximizes the fit, but constraints the parameter values to be non-negative.
This is the most appropriate fitting method for weighting RDMs, because weighting features in an underlying representation can only add to
the distances between points not subtract from them.

Regression-based methods
^^^^^^^^^^^^^^^^^^^^^^^^
For correlation and cosine distances and weighted models,  ``rsatoolbox.fitter.fit_regress`` provides more reliable and faster solutions
based on linear algebra. Being specialized this method works only for the 'cosine', 'corr', 'cosine_cov' and 'corr_cov'
comparison methods.

Analogously, ``rsatoolbox.fitter.fit_regress_nn`` provides a method for non-negative fits of such models.


Calling the fitting function directly
-------------------------------------
.. _modelfit:

All fitting methods take the following inputs: A ``model`` object, ``data``, a data RDMs object to fit to, ``method``, a string that defines which similarity
measure is optimized, and a ``pattern_idx``, ``pattern_descriptor`` combination that defines which subset of the RDM the provided
data RDMs correspond to. Additionally, they might take a ``ridge_weight``, which adds a penalty on the squared norm of the weights vector to the objective
and ``sigma_k``, which is required for adjusting the whitened correlation and cosine similarity measures to dependent measurements.

For simple fitting of a single model simply apply the fit function to the model and data as shown in the following example:

.. code-block:: python

    import rsatoolbox
    # generate 2 random model RDMs of 10 conditions
    model_features = [rsatoolbox.data.Dataset(np.random.rand(10, 7)) for i in range(2)]
    model_rdms = rsatoolbox.rdm.calc_rdm(model_features)
    model = rsatoolbox.model.ModelWeighted('test', model_rdms)

    # generate 5 random data RDMs of 10 conditions
    data = [rsatoolbox.data.Dataset(np.random.rand(10, 7)) for i in range(5)]
    data_rdms = rsatoolbox.rdm.calc_rdm(data)

    # fit model to group data to maximize cosine similarity using its default fitter
    theta = model.fit(data_rdms, method='cosine')

    # explicitly use the fit_optimize function to do the fit
    theta2 = rsatoolbox.model.fitter.fit_optimize(model, data_rdms, method='cosine')

Using the Fitter object
-----------------------

To provide an object, which fixes some parameters of the fitting function, rsatoolbox provides the fitter object. This type of object
is defined as ``rsatoolbox.model.fitter.Fitter`` and takes a fitting function and additional keyword arguments as inputs.
It then behaves as the original fitting function, with defaults changed to given keyword arguments.

To create a fitting function which sets the ``ridge_weight`` to 1 by default you could use the following code for example:

.. code-block:: python

    # fix some parameter of the function by using a fitter object:
    fitter = rsatoolbox.model.fitter.Fitter(rsatoolbox.model.fit_optimize, ridge_weight=1)
    theta3 = fitter(model, data_rdms, method='cosine')

Observe that this does indeed slightly change the fitted parameters compared to ``theta`` and ``theta2``

Both the fitting functions themselves and the fitter objects can be used as inputs to the crossvalidation and bootstrap-crossvalidation
methods to change how models are fit to data.
