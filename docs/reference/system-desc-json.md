# `system_desc.json`

The hardware and software description of the system under test. **One per Pareto point**, since
policies PR #119 there is no shared per-system file.

!!! danger "Authored by you"
    Not written by the reference client. You author it and drop it into each run folder before
    upload. Authored in [step 5](../workflow/author-disclosures.md).

The checker verifies that every point of a curve describes the **same** system
(`system-description-consistency`).

## Identity and classification

| Field | Description |
|---|---|
| `division` | `Standardized`, `Serviced` or `RDI` |
| `system_name` | Your string describing the system under test |
| `shortened_system_name` | Shortened name, **at most 20 characters** |
| `system_availability_status` | `Available`, `Preview` or `RDI` at submission time |
| `model_name` | Benchmark model name — must match the round's supported model list |
| `max_supported_concurrency` | Your declared `C_max` |
| `endpoint_url` | URL or description of the endpoint under test |

## Scale and topology

| Field | Description |
|---|---|
| `system_size` | Accelerators per node type, e.g. `"72 accelerators + 144 accelerators"` for a two-node-type system |
| `system_node_ensemble_count` | How many unique hardware+software combinations are in the system |
| `system_node_ensemble_total` | Total nodes — the sum of all `number_of_nodes` |
| `system_node_ensemble_id` | Identifies a unique node type within the system |
| `number_of_nodes` | How many nodes of this type |

## Host

| Field | Description |
|---|---|
| `host_processor_model_name` | Host processor model |
| `host_processors_per_node` | Host processors per node |
| `host_processor_core_count` | CPU cores per processor — optional, but at least one of core/vCPU count must be present |
| `host_processor_vcpu_count` | vCPUs per processor — same condition |
| `host_memory_capacity` | **Total** memory for all host processors, not per-processor |
| `host_memory_configuration` | DIMM count, memory type (DDR5, LPDDR4) and speed |
| `host_network_card_count` | Number and type of networking cards, with speeds |
| `host_networking` | Protocol — InfiniBand, Ethernet |
| `host_storage_capacity` | Total storage for the node |
| `host_storage_type` | Storage type |

## Accelerator

| Field | Description |
|---|---|
| `accelerator_model_name` | Accelerator model |
| `accelerators_per_node` | Accelerators per node |
| `accelerator_memory_capacity` | Memory per accelerator |
| `accelerator_memory_type` | Accelerator memory type |
| `accelerator_host_interconnect` | Link between accelerator and host processors |
| `accelerator_interconnect` | Link between accelerators, where `accelerators_per_node > 1` |

## Software

| Field | Description |
|---|---|
| `serving_framework` | SGLang, vLLM, etc. |
| `inference_backend` | Vendor stack components |
| `driver` | Driver and version for any accelerators |
| `operating_system` | OS |
| `filesystem` | Filesystem |
| `container_link` | Link to the submission container |
| `other_software_stack` | Other performance-relevant software, free-form |
| `sw_notes` | Supplementary software notes, free-form |

## Parallelism and configuration

| Field | Description |
|---|---|
| `tensor_parallel` | TP=N splits weight matrices N ways; each partition holds 1/N of each layer. Attention head count generally must divide by N. TP=1 means no partitioning |
| `expert_parallel` | EP=N splits experts into N groups on different accelerators, routing tokens to the right group. MoE models only. EP=1 means no partitioning |
| `pipeline_parallel` | PP=N splits layers into N sequential stages; an inference passes through all stages. PP=1 means no partitioning |
| `data_parallel` | DP=N replicates the model N times and distributes requests. DP=1 means no replication |
| `disaggregated` | Whether the system is disaggregated (`> 1`) |
| `batch` | Maximum batch size |
| `node_config` | Configuration of nodes/processors, in enough detail to reproduce the submission |
| `config_summary` | Concatenation of `disaggregated`, `tensor_parallel`, `pipeline_parallel`, `expert_parallel`, `data_parallel` where each is > 1, plus `config_summary_notes` |
| `config_summary_notes` | Free-form, for anything the concatenated fields miss |

!!! note "`link_config` was removed"
    Policies commit `b4ab404` dropped `link_config` from the §8.2 field table, but the template in
    §8.2.1 still carries it, so it's still shown in the template below. Leave it empty. Tracked as
    **B7**.

## Other

| Field | Description |
|---|---|
| `cooling` | Liquid, air-only, or passive |
| `other_hardware` | Other performance-relevant hardware, free-form |
| `hw_notes` | Supplementary hardware notes, free-form |
| `tps_utilization` | `reported_system_tps / max(reported_system_tps across all runs)` |

!!! warning "`tps_utilization` is computed, not chosen"
    The checker recomputes it against your own curve (`tps-utilization`). You cannot fill it in until
    every point has run.

!!! question "`disaggregated` is truncated upstream"
    The field definition in the source data dictionary is cut off mid-sentence and carries an
    upstream TODO to verify the full text with the dictionary owner. Confirm the intended semantics
    before relying on it.

## Template

```json
{
  "division": "",
  "system_name": "",
  "shortened_system_name": "",
  "system_availability_status": "",
  "system_size": "",
  "system_node_ensemble_count": 0,
  "system_node_ensemble_total": 0,
  "endpoint_url": "",
  "serving_framework": "",
  "node_types": [
    {
      "system_node_ensemble_id": 0,
      "number_of_nodes": 0,
      "host_processor_model_name": "",
      "host_processors_per_node": 0,
      "host_processor_core_count": 0,
      "host_processor_vcpu_count": 0,
      "host_memory_capacity": "",
      "host_memory_configuration": "",
      "accelerator_info": [
        {
          "accelerator_model_name": "",
          "accelerators_per_node": 0,
          "accelerator_memory_capacity": "",
          "accelerator_memory_type": "",
          "accelerator_interconnect": "",
          "accelerator_host_interconnect": ""
        }
      ],
      "host_network_card_count": "",
      "host_networking": "",
      "host_storage_capacity": "",
      "host_storage_type": "",
      "other_hardware": "",
      "cooling": "",
      "hw_notes": "",
      "inference_backend": "",
      "driver": "",
      "operating_system": "",
      "filesystem": "",
      "container_link": "",
      "other_software_stack": "",
      "sw_notes": ""
    }
  ],
  "node_config": "",
  "disaggregated": 0,
  "expert_parallel": 0,
  "tensor_parallel": 0,
  "pipeline_parallel": 0,
  "data_parallel": 0,
  "batch": 0,
  "config_summary": "",
  "config_summary_notes": "",
  "link_config": "",
  "tps_utilization": 0
}
```

## Checker rules that read this file

| Rule | Checks |
|---|---|
| `system-description-present` | Every point has a `system_desc.json` |
| `system-description-valid` | It parses against the `SystemDescription` schema |
| `system-description-consistency` | Every point of a curve describes the same system |
| `model-name-valid` | `model_name` is one of the round's supported models |
| `model-name-consistency` | It matches the results directory name |
| `max-concurrency-declared` | `max_supported_concurrency` present and > 32 |
| `tps-utilization` | Equals `system_tps / max(system_tps)` over the point's own curve |

!!! note "Field name drift"
    Most of the rules now say `system_desc.json`, but the §9.1 *Max concurrency declared* row and
    the Submission Rules still say `system_desc_id.json`, and the result-ID definition refers to a
    `benchmark_model` field. The tooling uses `system_desc.json` and `model_name`, and that's what
    the checker reads.

Provisioned power is **not** in this file. It goes in a separate, per-system
[`system_power.json`](system-power-json.md).

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
