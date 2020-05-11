Here are a few common examples...

## Read-based analysis

### Metaphlan

The workflow runs the recently released version 3 of [Metaphlan](https://github.com/biobakery/MetaPhlAn).

**Config:**
```yaml
metaphlan: True
metaphlan_index: "mpa_v30_CHOCOPhlAn_201901"
metaphlan_plot_rank: "genus"
```

**Command:**

```bash
snakemake --use-conda --config config.yaml -j 4 -p metaphlan_classify
```