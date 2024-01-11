# Organization info

## Repositories

- Toolbox
    - [dbpd toolbox](https://github.com/biomarkersParkinson/dbpd-toolbox): the main toolbox for data processing outputing scores indicating the progression of Parkinson's
- Project documentation
    - [docs](https://github.com/biomarkersParkinson/docs): for general documentation.
- Data handling (input, output, conversion, decryption, ...):
    - [tsdf](https://github.com/biomarkersParkinson/tsdf): official Python package for managing tsdf format.
    - [ppp](https://github.com/biomarkersParkinson/ppp): loading utilities for the _ppp_ dataset.
    - [gait](https://github.com/biomarkersParkinson/gait): loading utilities for the _pdathome_ dataset.
    - [TimeStreamDB](https://github.com/biomarkersParkinson/TimeStreamDB): Max's code for data formatting.
    - [pep-download](https://github.com/biomarkersParkinson/pep-download): Peter's data acquisition scripts.
    - [parkio](https://github.com/biomarkersParkinson/parkio): input/check/output of time series.

## Architecture

### Input

Although the inputs may differ in format, we expect them to contain time series information. On a per-patient basis, this can be read as an array where the first column contains the times, and the rest of the columns contain the corresponding measured states (such as accelerations, gyroscopic data, light intensity, ...):

| Times | Accel x   | Accel y   | ... |
|-------|-----------|-----------|-----|
| 0     | `<float>` | `<float>` | ... |
| 0.1   | `<float>` | `<float>` | ... |
| 0.2   | `<float>` | `<float>` | ... |
| 0.3   | `<float>` | `<float>` | ... |

To get those time series in a neat, usable way, a parsing and preprocessing workflow is needed for each data format:

```mermaid
graph TD;
    Input[("Raw data")] --> Parser --> Output[/Time series/]
```

### Desired output

Our desired output is a table containing different scores indicating the progression of Parkinson's. Notice that we aggregate them at a much longer scale than the devices' resolutions. The intuitive reason for doing this is that to witness significant progress in Parkinson's disease we need to wait weeks instead of milliseconds.

| Week | Gait score | Tremor score | ... |
|------|------------|--------------|-----|
| 1    | `<float>`  | `<float>`    | ... |
| 2    | `<float>`  | `<float>`    | ... |
| 3    | `<float>`  | `<float>`    | ... |
| 4    | `<float>`  | `<float>`    | ... |

Our proposed workflow to get there is the following:

![Pipelines](https://github.com/biomarkersParkinson/docs/blob/main/architecture/pipeline-architecture.drawio.png)

See an alternative illustration:

```mermaid
graph TD;

    subgraph specific context of use
         Input["Raw acc, gyr & ppg time series "]
    end

    subgraph gravity
         Gravity["Gravity filtering"]
    end

    subgraph gait
        Gait["Gait detection"]
        Cleangait["Detection of other activities"]
        Armswing["Arm swing quantification"]
        b["Weekly aggregation"]
    end

    subgraph tremor
        ArmActivity["Arm activity"]
        Tremor["Tremor detection"]
        c["Weekly aggregation"]
        TremorQuant["Tremor quantification"]
    end

    subgraph heart-rate variability
        Filter["Artefact detection"]
        HRstat["Global HR statistics"]
        HRvarex["HR exercise variability"]
        HRvarnight["Nighttime HR variability"]
        d["Weekly aggregation"]
    end

    Input --> Gravity --> Gait --> Cleangait --> Armswing --> b --> Scores[/Digital biomarkers/]

    Gait .-> Tremor
    Gravity --> ArmActivity--> Tremor --> c --> Scores
    Tremor --> TremorQuant --> c

    Input --> Filter
    Gait .-> HRvarex
    Filter --> HRvarex --> d
    Filter --> HRstat --> d --> Scores
    Filter --> HRvarnight --> d
```

## References

- [TSDF](https://arxiv.org/abs/2211.11294): a format standard for digital biosensor data
- [mcfly](https://github.com/NLeSC/mcfly): a deep-learning tool for time series classification created by the Netherlands eScience Center.
    - See [tutorial](https://blog.esciencecenter.nl/mcfly-an-easy-to-use-tool-for-deep-learning-for-time-series-classification-b2ee6b9419c2).
