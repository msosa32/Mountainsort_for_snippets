Running Mountainsort on spike snippets

This is applicable if you have saved the filtered waveforms of your spikes (aka “snippets” aka “clips”), rather than continuously recorded broadband data.
Convert waveforms to mda format:
Concatenate the waveforms horizontally with zeros padded in between.
For example, if your snippet data is saved in a 4x40xN Matlab array, where:
4 = number of channels (a tetrode)
40 = number of samples in the snippet
N = number of events
Concatenate with zeros(size(snippet,2)) between snippets such that you end up with a 2D matrix of 4 x (2*40*N)
The rows must be the channels of the ntrode
Zeroes can be any length but should be at least as long as a snippet
The padding is important so that when Mountainsort detects events and makes its own clips, there will not be overlap between the waveforms of neighboring spikes!
Convert this matrix to mda format using writemda()
Next, make a vector of pseudo event times for Mountainsort to use as the peak timestamps of each event.
Find the maximum amplitude across all channels per event.
Find the column index of that peak on that channel.
In other words, there will be 1 index per event, and this will correspond to the channel with the largest peak (even if the peaks on the other channels fall at slightly different indices).
This should be a 1D vector of indices into your 2D matrix of snippets.
To do all of this, look at Mari’s Matlab function waves2mda as an example.
Will create two files in a .mda directory:
raw.nt<num>.mda - the mda of your snippets per ntrode
event_times.nt<num>.mda - the indices of the peak for each event per ntrode

Run setup javascript
Same as for continuous data - create datasets.txt, pipelines.txt, raw.mda.prv, and params.json. Run in the parent directory above the .mda directory.

Run mountainsort
Sorting parameters:
Must manually pass in --prescribed_event_times=/path/to/event_times into sort after the pipeline call
Since each ntrode has its own event_times, you could do this at the command line or each ntrode could have an entry in pipelines.txt
In pipelines.txt:
--freq_min and --freq_max should be the bandpass used to filter your snippets during recording. Mountainsort will re-filter and you don’t want this to change the waveforms!
--detect_sign and --detect_interval_msec should match spike detection during recording. Although ms can detect spikes closer together from continuous data, you don’t want it detecting double spikes from your snippets.
--mask_out_artifacts=true to exclude noise artifacts (can also set the threshold)
Run sorting
Example: 
kron-run ms2 --prescribed_event_times=/datadir/dataset.mda/event_times.nt13.mda nt13
Curate clusters in mountainview
Firing rates and duration given in cluster metrics will be INCORRECT due to the padding.  
Autocorrelograms will also be useless for the same reason (you’ll never have anything at zero because you artificially added data points between spikes).
Snippets will still get whitened in the sorting process (this is useful!).
For curation, I suggest looking at:
Firing events (filtered)
Peak amplitude features (filtered)
Discrim histograms (whitened)
PCA features (whitened)
Amplitude histograms (whitened)