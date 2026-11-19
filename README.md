# Beacon Frame Dataset for Rogue Access Point Detection

This dataset contains IEEE 802.11 beacon frame data for research on Rogue Access Point (RAP) detection. It includes one baseline of legitimate access points and six evaluation files. The evaluation files cover two locations and three spoofing scenarios.

## Privacy and Anonymization

All SSIDs and BSSIDs are pseudonymized with a one-way hash.

- SSIDs use the format `SSID_<12 hex characters>`.
- BSSIDs use the format `AP_<12 hex characters>`.
- The same real value always maps to the same pseudonym in every file. Relations between files are therefore preserved.
- No raw MAC addresses or network names are included.
- The exact capture locations are not disclosed.

## Repository Structure

```
.
├── fingerprint_legit.json
├── eval_data_Campus_Th_0.15__SSID.json
├── eval_data_Campus_Th_0.15__SSID_RSN_Channel.json
├── eval_data_Campus_Th_0.15__SSID_RSN_Channel_RSSI.json
├── eval_data_Residence_Th_0.15__SSID.json
├── eval_data_Residence_Th_0.15__SSID_RSN_Channel.json
└── eval_data_Residence_Th_0.15__SSID_RSN_Channel_RSSI.json
```

## Data Collection

Beacon frames were captured on Linux with a wireless adapter in monitor mode. The frames were parsed with Python and Scapy.

Data was collected at two locations.

- **Campus.** A university environment with many access points.
- **Residence.** A residential environment with fewer access points.

The baseline was captured in a separate session from the evaluation data. Each evaluation file also comes from its own capture session. The sessions differ in spoofing scenario and recording time. For this reason, the row order and the set of observed access points differ between files of the same location.

## Spoofing Scenarios

Each evaluation file represents one spoofing scenario. The suffix of the file name lists the attributes that the rogue access point copied from the legitimate one.

| File suffix | Scenario | Attributes copied by the rogue AP |
|---|---|---|
| `SSID` | Basic | SSID only. |
| `SSID_RSN_Channel` | Intermediate | SSID, security type, and channel. |
| `SSID_RSN_Channel_RSSI` | Advanced | SSID, security type, channel, and a signal level close to the legitimate AP. |

`Th_0.15` in the file name is the detection threshold used to produce the evaluation output. See the paper for details.

## Baseline File

`fingerprint_legit.json` lists the legitimate access points. It contains 57 access points. Of these, 48 come from Campus and 9 come from Residence.

The file is a JSON object. Each key is a pseudonymized BSSID. Each value holds the fingerprint of that access point.

| Field | Type | Description |
|---|---|---|
| `ssid` | string | Pseudonymized network name. |
| `avg_rssi` | float | Mean signal strength in dBm during baseline capture. |
| `samples` | int | Number of beacon frames used to build the fingerprint. |
| `supported_rates` | list of float | Supported data rates in Mbps. |
| `rsn` | string | Security type, such as `WPA2-CCMP` or `OPEN`. |
| `vendor_ies` | list of string | OUIs of the vendor-specific Information Elements, in order of appearance. |
| `beacon_interval` | int | Beacon interval in time units (1 TU = 1024 µs). |
| `channel` | int | Operating channel. |
| `privacy` | bool | Privacy bit of the Capability Information field. |
| `ess` | bool | ESS bit of the Capability Information field. |

Example:

```json
"AP_88c42e68ba2f": {
  "ssid": "SSID_29589366dc77",
  "avg_rssi": -90.67,
  "samples": 45,
  "supported_rates": [1.0, 2.0, 5.5, 6.0, 9.0, 11.0, 12.0, 18.0, 24.0, 36.0, 48.0, 54.0],
  "rsn": "WPA2-CCMP",
  "vendor_ies": ["0050F2", "001018", "0050F2"],
  "beacon_interval": 100,
  "channel": 1,
  "privacy": true,
  "ess": true
}
```

## Evaluation Files

Each evaluation file is a JSON array. Each element is one received beacon frame.

| Field | Type | Description |
|---|---|---|
| `ssid` | string | Pseudonymized network name. |
| `bssid` | string | Pseudonymized BSSID. |
| `rssi` | float | Received signal strength in dBm. |
| `rsn` | string | Security type. |
| `channel` | int | Channel on which the frame was received. |
| `confidence` | float | Confidence score produced by the detector. |
| `fp_score` | float | Fingerprint similarity score against the baseline. |
| `match_type` | string | Result of matching against the baseline. See below. |
| `label` | string | Detector output. See below. |

| `match_type` | Meaning | `label` |
|---|---|---|
| `exact` | The BSSID exists in the baseline and the SSID matches. | `LEGIT` |
| `ssid_group` | The SSID exists in the baseline, but the BSSID does not. | `RAP` |
| `unknown` | The SSID does not exist in the baseline. | `UNKNOWN` |

## Ground Truth

The `label` field is the detector output. It is not the ground truth.

Each evaluation file contains exactly one rogue access point. Its BSSID is `AP_e980d9b2a66f` in all six files. The same rogue device was used at both locations. It imitated a different SSID at each location.

| Location | Rogue BSSID | Imitated SSID |
|---|---|---|
| Campus | `AP_e980d9b2a66f` | `SSID_98c7226eeaa9` |
| Residence | `AP_e980d9b2a66f` | `SSID_3a0cac043629` |

Rows with `bssid == "AP_e980d9b2a66f"` are rogue. All other rows are legitimate or unrelated access points.

## Dataset Size

| File | Rows | Unique APs | Rogue rows |
|---|---|---|---|
| Campus, Basic | 1600 | 47 | 83 |
| Campus, Intermediate | 1600 | 70 | 69 |
| Campus, Advanced | 1600 | 71 | 52 |
| Residence, Basic | 800 | 23 | 100 |
| Residence, Intermediate | 800 | 22 | 100 |
| Residence, Advanced | 800 | 17 | 100 |

## Usage Example

```python
import json

with open("fingerprint_legit.json") as f:
    baseline = json.load(f)

with open("eval_data_Campus_Th_0.15__SSID.json") as f:
    frames = json.load(f)

ROGUE = "AP_e980d9b2a66f"
for r in frames:
    r["is_rogue"] = r["bssid"] == ROGUE

print(len(baseline), "baseline APs")
print(len(frames), "frames,", sum(r["is_rogue"] for r in frames), "from the rogue AP")
```

## License

This dataset is released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license. You may share and adapt it for any purpose. You must give appropriate credit.

## Authors

- Zikri Zulfa Azhim, Department of Electrical Engineering, Universitas Indonesia
- Alfan Presekal, Department of Electrical Engineering, Universitas Indonesia
- Riri Fitri Sari, Department of Electrical Engineering, Universitas Indonesia
