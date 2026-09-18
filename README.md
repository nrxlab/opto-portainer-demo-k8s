# Opto22 & Portainer demo using groov Manage REST API

Containerized collector for direct groov RIO I/O reads through the groov Manage REST API, with MQTT publishing to a minimal Mosquitto broker.

It polls the packed module endpoints:

```text
GET /manage/api/v1/io/local/modules/<module>/analog/values?channels=8
GET /manage/api/v1/io/local/modules/<module>/digital/values
```

It discovers channel configuration once at startup and publishes a compact name-keyed JSON payload.

# deployments

Clone this repository
```
git clone https://github.com/nrxlab/opto-portainer-demo-k8s.git
```

## configure

Copy the template and fill in your device's values:

```bash
cp secrets-template.yaml secrets.yaml
```

apply to node or cluster directly:
```
kubectl -f apply secrets.yaml
```

The secrets can also be created directly within the Portainer user interface by using the "Create from code" action utilizing the secrets-template.yaml from this repository and then editing created secrets within Portainer directly to update with the privileged information.

Supported output modes:

- `stdout`
- `mqtt`
- `both`

The deployment manifest has commented options to support the following:
- namespace and secret definitions for a single file deployment
- Xiid Stlink sidecar to the mosquitto broker to enable secure remote access (utilize the deployment-xiid.yaml)
- nodePort option to enable mosquitto broker access from a specific node/host

## subscribe to the broker

From another shell:

```bash
docker compose exec mosquitto mosquitto_sub -t 'opto/rio/#' -v
```

Using `mosquitto-clients` if installed:

```bash
mosquitto_sub -h localhost -p 1883 -t 'opto/rio/#' -v
```

## payload shape

```json
{
  "timestamp": "2026-09-03T14:15:32.804459972Z",
  "host": "10.0.2.55",
  "device": "local",
  "module": 0,
  "fields": {
    "di-button01": {"channel": 0, "kind": "digital", "value": true, "qualityError": false, "channelType": "0x50000079"},
    "ai-potentiometer": {"channel": 2, "kind": "analog", "value": 1.91, "unit": "V", "qualityError": false, "channelType": "0x60000018"},
    "ai-ambient-temp": {"channel": 3, "kind": "analog", "value": 25.9, "unit": "Degrees C", "qualityError": false, "channelType": "0x60000031"}
  }
}
```

Set `OPTO_INCLUDE_RAW=true` to also include raw packed analog/digital arrays.
