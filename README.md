# Automation Native Campus Network Design

An example of how an enterprise campus network should be designed when automation is part of the design, not an add-on. The design is expressed as structured, schema-validated data. Device configuration is rendered from that data, deployed by Ansible and validated against it.

## Start Here: [Campus Network Design](Campus%20Network%20Design.md)

[Campus Network Design](Campus%20Network%20Design.md) is the main document. It explains the approach and then applies it to a 2,000 user, 10-floor campus for a company called ABC:

| Section of Campus Network Design | What it covers |
|---|---|
| [Executive Summary](Campus%20Network%20Design.md#automation-native-network-design-explain---executive-summary) | The six elements of an automation-native design, and why the classic design approach drifts. |
| [Core Concept Flow](Campus%20Network%20Design.md#core-concept-flow) | schema + values = data model, data model + template = rendered config, playbook + rendered config = deployed config, validation = data model vs deployed config. |
| [Campus Network Design](Campus%20Network%20Design.md#campus-network-design) | Business and technical requirements, scope, topology, control plane and overlay, services, platforms, resiliency and capacity. |
| [Data Models](Campus%20Network%20Design.md#data-models) | The design-driven models and the device role models, each with its schema and template. |
| [Design Validation](Campus%20Network%20Design.md#design-validation) | Checking that the rendered configuration matches the data model. |
| [Design Deployment](Campus%20Network%20Design.md#design-deployment-aka-low-level-design) | The step-by-step build, using the playbooks in this repo. |
| [Config Output](Campus%20Network%20Design.md#config-output) and [Telemetry Output](Campus%20Network%20Design.md#telemetry-output) | What the pipeline produces. |

## How the Repo Maps to the Flow

| Flow step | Folder | What is in it |
|---|---|---|
| schema | [`schemas/`](schemas/) | A JSON Schema for every data model, one `.schema.json` per model. |
| values = data model | [`models/`](models/) | The data models as YAML: physical topology, management network, logical topology, endpoint service, telemetry, and the three device role models (WAN edge, core and aggregation, access). |
| template | [`templates/`](templates/) | Jinja2 templates: one per platform baseline (Catalyst 8000, Nexus 93240, Catalyst 9000), and one each for physical topology, logical topology, endpoint service and telemetry. |
| rendered config | [`config/`](config/) | The rendered configuration for each device. |
| playbook | [`ansible_resources/`](ansible_resources/) | The Ansible playbooks, inventory, filter plugin and collection requirements. |
| validation | [`scripts/`](scripts/) and [`validation_report.md`](validation_report.md) | Scripts that render and validate configs, and the report they produce. |

## Playbooks

Run from `ansible_resources/`. `site.yml` is the master playbook for building a site. It runs, in order:

1. [`00_validate_render.yml`](ansible_resources/playbooks/00_validate_render.yml) - pre-flight: checks the rendered CLI against the data model.
2. [`01_baseline_build.yml`](ansible_resources/playbooks/01_baseline_build.yml) - platform baseline.
3. [`02_physical_topology.yml`](ansible_resources/playbooks/02_physical_topology.yml) - hostnames, interfaces, point-to-point connectivity.
4. [`03_logical_topology.yml`](ansible_resources/playbooks/03_logical_topology.yml) - loopbacks, addressing, BGP and the EVPN overlay.
5. [`04_streaming_telemetry.yml`](ansible_resources/playbooks/04_streaming_telemetry.yml) - streaming telemetry subscriptions.

[`bau_endpoint_provisioning.yml`](ansible_resources/playbooks/bau_endpoint_provisioning.yml) is separate. It provisions an access port as a business-as-usual change, and is not part of the site build. [`set_endpoint_port.py`](scripts/set_endpoint_port.py) records such a change in the data model so the next render includes it.

The commands to run them are in [Design Deployment](Campus%20Network%20Design.md#design-deployment-aka-low-level-design).

## Scripts

| Script | What it does |
|---|---|
| [`render_configs.py`](scripts/render_configs.py) | Renders the full per-device configuration by chaining the platform baseline, physical topology, logical topology, telemetry and endpoint service templates. |
| [`validate_configs.py`](scripts/validate_configs.py) | Checks each rendered config against the data model, using the same lookup functions the templates use. Produces [`validation_report.md`](validation_report.md). |
| [`generate_schemas.py`](scripts/generate_schemas.py) | Generates a JSON Schema for every data model in `models/`. |
| [`set_endpoint_port.py`](scripts/set_endpoint_port.py) | Records one switchport change in the endpoint service data model. |

## Other Files

- [`diagram/`](diagram/) - The diagrams used in Campus Network Design: physical, logical, management topology, data model relationships and the automation-native pipeline.
- [`Rack_Patching_Record.md`](Rack_Patching_Record.md) - Rack elevation and patching matrix, submitted to the facility team for the physical build.
