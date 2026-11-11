# Data and Feature Archives

Large precomputed feature tensors are distributed outside Git.

Download the final archives from:

[MMML final feature archives](https://drive.google.com/drive/folders/1Y5hwU1WikPbT5pzhIL0ocmJcf9MWE1mv)

Expected archive names:

```text
MITNN_MOF_final_features.zip
MITNN_LD50_final_features.zip
```

The archive file names keep the earlier `MITNN_` prefix.

Extract both archives from the repository root:

```bash
unzip MITNN_MOF_final_features.zip
unzip MITNN_LD50_final_features.zip
```

Expected directories after extraction:

```text
data/
├── mof/
│   ├── 2STD/
│   └── features/
│       ├── PH/
│       ├── PL/
│       ├── CA/
│       └── FPRC/
└── ld50/
    ├── LD50_train.csv
    ├── LD50_test.csv
    └── topology_features/
        ├── PH/
        ├── PL/
        ├── CA/
        ├── EIC/
        └── FPRC/
```

