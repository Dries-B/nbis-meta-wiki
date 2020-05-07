The workflow can be configured using a configuration file in `yaml` format. These are the configuration parameters and their default values

## Paths
```yaml
# Sample information file (required)
sample_list: "samples/example_sample_list.tsv"
# Work directory for workflow
workdir: .
# Main results directory
results_path: results
# Path to store report files
report_path: results/report
# Intermediate results path
intermediate_path: results/intermediate
# Temporary path
temp_path: temp
# Local storage path
scratch_path: temp
# Path to store resource files (databases etc)
resource_path: resources
```