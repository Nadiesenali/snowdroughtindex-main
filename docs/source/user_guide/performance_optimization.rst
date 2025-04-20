Performance Optimization
======================

This guide provides strategies for optimizing the performance of the Snow Drought Index package when working with large datasets.

Introduction
-----------

The Snow Drought Index package includes several performance optimization features that can significantly improve execution speed and memory efficiency when working with large datasets. This guide covers:

1. Data handling optimization techniques
2. Computation optimization strategies
3. Parallel processing capabilities
4. Memory management approaches
5. Benchmarking and profiling tools

Data Handling Optimization
-------------------------

Lazy Loading
^^^^^^^^^^^

The package supports lazy loading of data through xarray's dask integration, which allows you to work with datasets larger than memory:

.. code-block:: python

    from snowdroughtindex.core.dataset import SWEDataset
    
    # Enable lazy loading when creating a SWEDataset object
    dataset = SWEDataset('path/to/large_swe_data.nc', lazy_loading=True)
    
    # Load data with chunking
    dataset.load_data(chunks={'time': 100, 'lat': 50, 'lon': 50})

Chunking Strategies
^^^^^^^^^^^^^^^^^

Proper chunking can significantly improve performance when working with large datasets:

.. code-block:: python

    # Optimize chunks for time series analysis (smaller chunks in time dimension)
    dataset.load_data(chunks={'time': 30, 'lat': 100, 'lon': 100})
    
    # Optimize chunks for spatial analysis (smaller chunks in spatial dimensions)
    dataset.load_data(chunks={'time': 365, 'lat': 20, 'lon': 20})
    
    # Auto-chunking based on available memory
    dataset.load_data(chunks='auto')

Data Filtering
^^^^^^^^^^^^

Filter data early in your workflow to reduce memory usage:

.. code-block:: python

    # Filter by time range
    dataset.filter_by_time(start_date='1980-01-01', end_date='2020-12-31')
    
    # Filter by region
    dataset.filter_by_bbox(lon_min=-125, lon_max=-115, lat_min=35, lat_max=45)
    
    # Filter by basin
    dataset.filter_by_basin('path/to/basin_shapefile.shp', basin_name='Basin Name')
    
    # Filter by elevation
    dataset.filter_by_elevation(min_elevation=1000, max_elevation=3000)

Computation Optimization
-----------------------

Vectorization
^^^^^^^^^^^

The package uses vectorized operations instead of loops wherever possible:

.. code-block:: python

    # Example of vectorized operation in custom analysis
    import numpy as np
    
    # Instead of:
    # for i in range(len(data)):
    #     result[i] = some_function(data[i])
    
    # Use vectorized operations:
    result = np.vectorize(some_function)(data)
    
    # Or even better, use NumPy's built-in vectorized functions:
    result = np.exp(data) / (1 + np.exp(data))  # Sigmoid function

Numba Acceleration
^^^^^^^^^^^^^^^^

For computationally intensive functions, the package uses Numba for just-in-time compilation:

.. code-block:: python

    from numba import jit
    import numpy as np
    
    # Example of using Numba for a custom function
    @jit(nopython=True, parallel=True)
    def custom_integration(data, axis=0):
        result = np.zeros(data.shape[0])
        for i in range(data.shape[0]):
            result[i] = np.trapz(data[i, :], axis=axis)
        return result
    
    # Use the accelerated function
    integrated_data = custom_integration(swe_data)

Caching
^^^^^^

The package implements result caching to avoid redundant calculations:

.. code-block:: python

    from snowdroughtindex.core.sswei_class import SSWEI
    
    # Create an SSWEI object with caching enabled
    sswei = SSWEI(dataset, enable_caching=True, cache_dir='./cache')
    
    # Calculate SSWEI (results will be cached)
    sswei.calculate()
    
    # Subsequent calls with the same parameters will use cached results
    sswei.calculate()  # Uses cached results if available

Parallel Processing
-----------------

Multi-threading
^^^^^^^^^^^^^

The package supports multi-threading for I/O-bound operations:

.. code-block:: python

    from snowdroughtindex.core.dataset import SWEDataset
    
    # Create a SWEDataset object with multi-threading enabled
    dataset = SWEDataset('path/to/swe_data.nc', parallel=True, n_jobs=4)
    
    # Load data with multi-threading
    dataset.load_data(parallel=True, n_jobs=4)

Multi-processing
^^^^^^^^^^^^^^

For CPU-bound operations, the package supports multi-processing:

.. code-block:: python

    # Fill gaps with multi-processing
    dataset.fill_gaps(method='linear', parallel=True, n_jobs=4)
    
    # Calculate SSWEI with multi-processing
    sswei = SSWEI(dataset)
    sswei.calculate(parallel=True, n_jobs=4)
    
    # Analyze drought conditions with multi-processing
    analysis = DroughtAnalysis(sswei)
    analysis.analyze_elevation_bands(parallel=True, n_jobs=4)

Dask Integration
^^^^^^^^^^^^^^

For distributed computing, the package integrates with Dask:

.. code-block:: python

    import dask.distributed as dd
    
    # Create a Dask client
    client = dd.Client()  # Local cluster
    # Or connect to an existing cluster
    # client = dd.Client('scheduler-address:8786')
    
    # Create a SWEDataset object with Dask integration
    dataset = SWEDataset('path/to/swe_data.nc', dask_client=client)
    
    # Operations will be distributed across the Dask cluster
    dataset.fill_gaps(method='linear')
    
    # Close the client when done
    client.close()

Memory Management
---------------

Memory-Efficient Algorithms
^^^^^^^^^^^^^^^^^^^^^^^^^

The package implements memory-efficient algorithms for large datasets:

.. code-block:: python

    # Use memory-efficient gap filling
    dataset.fill_gaps(method='linear', memory_efficient=True)
    
    # Use memory-efficient SSWEI calculation
    sswei = SSWEI(dataset)
    sswei.calculate(memory_efficient=True)

Garbage Collection
^^^^^^^^^^^^^^^

Explicitly trigger garbage collection to free memory:

.. code-block:: python

    import gc
    
    # After completing a memory-intensive operation
    gc.collect()

Memory Monitoring
^^^^^^^^^^^^^^^

Monitor memory usage during execution:

.. code-block:: python

    from snowdroughtindex.utils.performance import memory_usage
    
    # Monitor memory usage of a function
    @memory_usage
    def process_data(dataset):
        # Process data
        return result
    
    # Call the monitored function
    result = process_data(dataset)
    
    # The decorator will print memory usage information

Benchmarking and Profiling
------------------------

Benchmarking Tools
^^^^^^^^^^^^^^^^

The package includes benchmarking tools to measure performance:

.. code-block:: python

    from snowdroughtindex.utils.performance import benchmark
    
    # Benchmark a function
    @benchmark
    def process_data(dataset):
        # Process data
        return result
    
    # Call the benchmarked function
    result = process_data(dataset)
    
    # The decorator will print execution time

Profiling
^^^^^^^^

Profile your code to identify bottlenecks:

.. code-block:: python

    import cProfile
    import pstats
    
    # Profile a function
    def profile_function(func, *args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        stats = pstats.Stats(profiler).sort_stats('cumtime')
        stats.print_stats(20)  # Print top 20 time-consuming functions
        return result
    
    # Profile a function call
    result = profile_function(dataset.fill_gaps, method='linear')

Performance Configuration
-----------------------

The package includes a performance configuration system:

.. code-block:: python

    from snowdroughtindex.core.configuration import Configuration
    
    # Create a configuration object
    config = Configuration()
    
    # Set performance parameters
    config.set_performance_params(
        parallel=True,
        n_jobs=4,
        lazy_loading=True,
        chunks={'time': 100, 'lat': 50, 'lon': 50},
        memory_efficient=True,
        enable_caching=True,
        cache_dir='./cache'
    )
    
    # Create a SWEDataset object with the configuration
    dataset = SWEDataset('path/to/swe_data.nc', config=config)

Best Practices
------------

Here are some best practices for optimizing performance:

1. **Filter Early**: Filter data as early as possible in your workflow to reduce memory usage.
2. **Choose Appropriate Chunking**: Optimize chunk sizes based on your analysis type (time series vs. spatial).
3. **Use Parallel Processing Wisely**: Enable parallel processing for computationally intensive operations, but be aware of overhead.
4. **Monitor Memory Usage**: Keep an eye on memory usage, especially when working with large datasets.
5. **Profile Your Code**: Identify bottlenecks and optimize the most time-consuming operations.
6. **Use Lazy Loading**: Enable lazy loading when working with datasets larger than memory.
7. **Implement Caching**: Cache intermediate results to avoid redundant calculations.
8. **Optimize I/O Operations**: Minimize disk I/O by loading data once and reusing it.

Example: Optimized Workflow
-------------------------

Here's an example of an optimized workflow for analyzing a large dataset:

.. code-block:: python

    import matplotlib.pyplot as plt
    from snowdroughtindex.core.dataset import SWEDataset
    from snowdroughtindex.core.sswei_class import SSWEI
    from snowdroughtindex.core.drought_analysis import DroughtAnalysis
    from snowdroughtindex.core.configuration import Configuration
    
    # Create a performance-optimized configuration
    config = Configuration()
    config.set_performance_params(
        parallel=True,
        n_jobs=4,
        lazy_loading=True,
        chunks={'time': 100, 'lat': 50, 'lon': 50},
        memory_efficient=True,
        enable_caching=True,
        cache_dir='./cache'
    )
    
    # Create a SWEDataset object with the optimized configuration
    dataset = SWEDataset('path/to/large_swe_data.nc', config=config)
    
    # Load and preprocess data
    dataset.load_data()
    
    # Filter data early to reduce memory usage
    dataset.filter_by_time(start_date='1980-01-01', end_date='2020-12-31')
    dataset.filter_by_basin('path/to/basin_shapefile.shp', basin_name='Basin Name')
    
    # Preprocess the filtered data
    dataset.preprocess()
    
    # Fill gaps with parallel processing
    dataset.fill_gaps(method='linear', parallel=True, n_jobs=4)
    
    # Create an SSWEI object with caching enabled
    sswei = SSWEI(dataset, enable_caching=True, cache_dir='./cache')
    
    # Calculate SSWEI with parallel processing
    sswei.calculate(parallel=True, n_jobs=4)
    
    # Classify drought conditions
    sswei.classify_drought()
    
    # Create a DroughtAnalysis object
    analysis = DroughtAnalysis(sswei)
    
    # Analyze drought conditions with parallel processing
    analysis.analyze_elevation_bands(
        elevation_breaks=[1000, 1500, 2000, 2500, 3000],
        elevation_data='path/to/elevation_data.nc',
        parallel=True,
        n_jobs=4
    )
    
    # Visualize results (this will trigger computation of lazy arrays)
    plt.figure(figsize=(10, 6))
    analysis.plot_elevation_analysis()
    plt.title('Drought Conditions by Elevation Band')
    plt.tight_layout()
    plt.show()
    
    # Clean up and free memory
    import gc
    gc.collect()

Conclusion
---------

By applying these performance optimization techniques, you can significantly improve the execution speed and memory efficiency of the Snow Drought Index package when working with large datasets. The package provides a flexible framework that allows you to tailor the optimization strategy to your specific needs and hardware capabilities.

For more information on performance optimization, refer to:

- :doc:`API Reference <../api/core>`
- :doc:`Configuration Guide <../user_guide/class_based_implementation>`
- :doc:`Example Notebooks <../user_guide/examples>`
