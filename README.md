# Dynamic Social Network Analysis Course at LVIV-DSSS-2018 

This is the material for the course about machine learning for dynamic
social network analysis held at the Lviv Data Science Summer School 2018.

## Hawkes

Here, we model recurring user activities with the help of Hawkes Processes.

There is only one python file with one unimplemented function (`sampleHawkes`)
in the file `simPointProcess.py`. The file may be run in an interactive console
and is self explanatory.

##### Requirements

  - cvxpy

## Redqueen

Here, we perform smart broadcasting with the help of stochastic optimal
control.

There is only one python file with one unimplemented function (``) in the
file ``. The file may be run in an interactive console.

##### Requirements

   - decorated_options: Installation instructions are at [musically-ut/decorated_options](https://github.com/musically-ut/decorated_options) or `pip install decorated_options`.

##### Execution

The code execution is detailed in the included IPython notebooks.

As an example, say if we have the following structure in our network of
broadcasters (i.e. sources) and followers (i.e. sinks), with `Source 1` being
the broadcaster we control:

```
   Source 1**      Source 2      Source 3
    +   +             +                 +
    |   |             |                 |
    |   +----------------------------+  |
    |                 |              |  |
    |    +------------+-----------+  |  |
    |    |            |           |  |  |
   +v----v-+      +---v---+      +v--v--v+
   |       |      |       |      |       |
   |       |      |       |      |       |
   |       |      |       |      |       |
   |       |      |       |      |       |
   |       |      |       |      |       |
   |       |      |       |      |       |
   +-------+      +-------+      +-------+
    Sink 1          Sink 2         Sink 3
```

This will be represented in the following way:

```python
simOpts = SimOpts(
   src_id = 1,
   end_time = 100, # When the simulations stop
   s = { 1: 1.0, 3: 1.0 }, # Weights of followers of source 1
   q=1.0, # Control parameter for RedQueen
   sink_ids = [1, 2, 3],
   other_sources = [
      ('Poisson2', { 'src_id': 2,
                    'seed': 42,
                    'rate': 10
                  }),
      ('Hawkes',  { 'src_id': 3,
                    'seed': 43,
                    'l_0': 10,
                    'alpha': 1.0,
                    'beta': 10.0
                  })
   ],
   edge_list=[(1, 1), (1, 3), 
              (2, 1), (2, 2), (2, 3),
              (3, 3)]
);
```

These `SimOpts` objects are immutable and can be used to create multiple simulations.

Then to run the simulation, we need to create a simulation `Manager` by instantiating `Source 1`
to be a kind of broadcaster, or by removing it altogether.

```python
manager = simOpts.create_manager_with_opt(seed=101)
# or 
manager = simOpts.create_manager_for_wall()
```

Finally, run the simulation by calling `.run_dynamic`:

```python
manager.run_dynamic()
```

Finally, the list of events can be extracted for further analysis:

```python
df = manager.state.get_dataframe()
```


The file `utils.py` contains some functions which can assist in calculation of
certain metrics:

```python
import redqueen.utils as U
perf_1 = U.time_in_top_k(df=df, K=1, sim_opts=simOpts)
perf_2 = U.average_rank(df=df, sim_opts=simOpts)
```
