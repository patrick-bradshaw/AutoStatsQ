# AutoStatsQ (-work in progress-)
Toolbox for automated station quality control for MT inversion 
Please contact me for further description and help: gesap@gfz-potsdam.de

- Catalog search for teleseismic events with uniform azimuthal coverage around array
- Download of data & metadata for these events + computation of synthetic data
- Relative gain factors in time domain (relative to reference station)
- Rayleigh wave polarization analysis for detection of sensor misorientations
- Comparison of obs. and synth. PSDs; determining frequency ranges suitable for MT inversion


Requirements
------------

- python3
- Seismology toolbox pyrocko: https://pyrocko.org/ (Heimann et al. 2017)
- To compute synthetic data a pre-calculated GF database can be downloaded using `fomosto download` from the pyrocko environment. http://kinherd.org:8080/gfws/static/stores/


Download and Installation
-------------------------

- cd into the folder where you want to do the installation
- git clone https://github.com/gesape/AutoStatsQ
- cd AutoStatsQ
- (sudo) python3 setup.py install

Basic commands
--------------

show basic commands/ help:

	autostatsq -h


generate an example config file:

	autostatsq --generate_config GENERATE_CONFIG

run 

	autostatsq --config name_of_config_file --run RUN


Settings in the config file
---------------------------

The default config file should look like this (without the comments):

```yaml
--- !autostatsq.config.AutoStatsQConfig
Settings:
- !autostatsq.config.GeneralSettings
  data_dir: /some/data/directory/
  list_station_lists: [/path/to/station-file/file.csv, /path/to/station-file/file.xml]

- !autostatsq.config.CatalogConfig
  search_events: true 
  # search gCMT catalog for events?

  use_local_catalog: false
  # or use a local (already downloaded) catalog? This option is needed if the toolbox is run 
  # in single steps, not all at once. 

  subset_of_local_catalog: true
  # Find a subset of the full catalog?

  use_local_subsets: false
  # Use local (already saved) subset instead?
  
  subset_fns: {}
  # if so, give here paths to subset-catalog-files: e.g. {'deep': 'catalog_deep_subset.txt',
  # 'shallow': 'catalog_shallow_subset.txt'}

  catalog_fn: catalog.txt
  # filename of catalog that is downloaded or already exists

  dist: 165.0
  # max. distance (degrees) for catalog search

  min_mag: 6.5
  max_mag: 8.5
  tmin_str: '2000-01-01 00:00:00'
  tmax_str: '2018-10-01 00:00:00'
  min_dist_km: 1000.0
  max_dist_km: 9999999.9  
  depth_options:
    deep: [25000, 1000000] # [m]
    shallow: [100, 40000] # [m]

  wedges_width: 15
  # backazimuthal step for subset generation

  mid_point: [46.98, 10.74]
  # estimate of midpoint of array

  median_ev_in_bin: false
  # ???


  ### catalog plotting options ###
  plot_catalog_all: false
  # plots entire catalog on a map

  plot_hist_wedges: false
  plot_wedges_vs_dist: false
  plot_wedges_vs_magn: false
  plot_dist_vs_magn: false
  # catalog statistics plot
  
  plot_catalog_subset: false
  # plots the subset(s) on a map

- !autostatsq.config.ArrTConfig
  calc_first_arr_t: true
  # Should first arrivals be computed?

  phase_select: P|p|P(cmb)P(icb)P(icb)p(cmb)p|P(cmb)Pv(icb)p(cmb)p|P(cmb)P<(icb)(cmb)p
  # which phases?

  calc_est_R: true
  # compute arrival time of Rayleigh waves for each station-event pair? 
  # (needed for orientation test)


- !autostatsq.config.CakeTTTGenerator
  # uses travel time tables instead of cake (faster, but settings more difficult)
  calc_ttt: false
  dir_ttt: /directory/to/save/traveltimes/
  earthmodel_id: prem-no-ocean.f
  tabulated_phases:
  - !pf.TPDef
    id: p
    definition: P,p
  dist_min: 1000.0
  dist_max: 9999.0
  dist_acc: 2.0
  s_depth_min: 10.0
  s_depth_max: 50.0
  s_acc: 2.0
  r_depth_min: 0.0
  r_depth_max: 0.0
  r_acc: 2.0
  t_acc: 3.0

- !autostatsq.config.MetaDataDownloadConfig
  # download of metadata and data

  download_data: false
  download_metadata: false
  use_downmeta: false
  components_download: HH*
  token:
    geofon: /home/gesap/Documents/AlpArray/download-waveforms/swathD/token.asc
  sites: [geofon, orfeus, iris]

  dt_start: 0.1
  # start time before origin time [h]
  dt_end: 1.5
  # end time after origin time [h]

- !autostatsq.config.RestDownRotConfig
  # restitution, downsampling and rotation of data
  rest_data: false
  freqlim: [0.005, 0.01, 0.2, 0.25]
  rotate_data: false
  deltat_down: 2

- !autostatsq.config.SynthDataConfig
  # computation of synthetic data
  make_syn_data: false
  engine_path: /path/to/GF_stores
  store_id: global_2s

- !autostatsq.config.GainfactorsConfig
  # settings for first test
  calc_gainfactors: false
  gain_factor_method:
  - reference_nsl
  - [GE, MATE]
  ### describe different methods
  fband:
    corner_hp: 0.01
    corner_lp: 0.2
    order: 4
  taper_xfrac: 0.25

  wdw_st_arr: 5
  wdw_sp_arr: 60
  # time window around P phase onset, start [s] before and end [s] after theo. arrival time

  phase_select: first(P|p|P(cmb)P(icb)P(icb)p(cmb)p|P(cmb)Pv(icb)p(cmb)p|P(cmb)P<(icb)(cmb)p)
  components: [Z, R, T]

  # plotting options
  plot_median_gain_on_map: false
  plot_allgains: false

- !autostatsq.config.PSDConfig
  # settings for PSD test
  calc_psd: false
  tinc: 600
  tpad: 200
  dt_start: 60
  dt_end: 1800
  n_poly: 25
  norm_factor: 50
  f_ign: 0.02
  only_first: true
  plot_psds: false
  plot_ratio_extra: false
  plot_m_rat: false
  plot_flat_ranges: false
  plt_neigh_ranges: false

- !autostatsq.config.OrientConfig
  # settings for orientation test
  orient_rayl: false
  bandpass: [3.0, 0.01, 0.05]
  start_before_ev: 30.0
  stop_after_ev: 480.0
  plot_heatmap: false
  plot_distr: false
  plot_orient_map_fromfile: false

- !autostatsq.config.maps
  # settings for all output maps
  map_size: [30.0, 30.0]
  pl_opt: [46, 11.75, 800000]
  pl_topo: false
```