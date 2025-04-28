# Viewing OpenShift Monitoring Dashboard Expressions as MarkDown

## Purpose

This guide describes how to extract and review Prometheus queries (`expr`) and panel metadata from OpenShift Monitoring dashboards stored as ConfigMaps in the `openshift-config-managed` namespace.

A list of Prometheus queries (`expr`) and associated metadata extracted from OpenShift Monitoring dashboards is tabulated in MarkDown format. 
The data in the table are obtained from `ConfigMap` resources labeled with `console.openshift.io/dashboard=true` from the `openshift-config-managed` namespace.

Each row shows:
- The source ConfigMap name (dashboard)
- The panel title
- The query expression (`expr`)
- A description, where available

This provides an easy-to-read overview of the monitoring queries that power the OpenShift web console dashboards.

For more information about OpenShift Monitoring, refer to the [OpenShift Monitoring Overview](https://docs.openshift.com/container-platform/4.10/monitoring/monitoring-overview.html).


> [!Tip]
> **TL;DR**  See example table:
> 👉 [guide-openshift-monitoring-dashboard-expressions-table.md](guide-openshift-monitoring-dashboard-expressions-table.md)

---

## Prerequisites

- OpenShift cluster access with permission to `oc get configmaps` in the `openshift-config-managed` namespace.
- Installed `jq` command-line tool (for JSON parsing).

---

## Quick Command

The following one-liner outputs a Markdown table listing all dashboard panels and their associated queries:

```bash
oc get configmaps -n openshift-config-managed -l console.openshift.io/dashboard=true -o json | jq -Cr '
  [
    "| OpenShift Monitoring Dashboard | Panel Title | Query | Description |",
    "|:---------|:------------|:------|:------------|"
  ]
  + (
    [
      .items[] |
      .metadata.name as $NAME |
      (.data[] | fromjson) as $DASHJSON |
      $DASHJSON.panels[]? as $panel |
      $panel.targets[]? as $target |
      "| \($NAME) | \( ($panel.title // "No Panel Title") ) | \( ($target.expr // "No Query Found") ) | \( ($panel.description // "" ) ) |"
    ]
  )
  | .[]
'
```

---

## Example Output

```markdown
| Dashboard | Panel Title | Query | Description |
|:---------|:------------|:------|:------------|
| grafana-dashboard-cluster | API Server Request Rate | sum(rate(apiserver_request_total[5m])) | Request rate across all API servers |
| grafana-dashboard-etcd | etcd Leader Changes | sum(rate(etcd_server_leader_changes_seen_total[5m])) | Number of times leadership changed in etcd |
| grafana-dashboard-node | CPU Usage | avg(node_cpu_seconds_total{mode!="idle"}) by (instance) | CPU usage excluding idle time |
```

(See full results in [guide-openshift-monitoring-dashboard-expressions-table.md](guide-openshift-monitoring-dashboard-expressions-table.md)).

---

## Notes

- Panels without a `title` are labeled as `No Panel Title`.
- Targets without an `expr` are labeled as `No Query Found`.
- Panel `description` is optional and may be empty.
- Only dashboards labeled with `console.openshift.io/dashboard=true` are included.

---

## Troubleshooting

- If you receive a `Forbidden` error, ensure your account has appropriate RBAC permissions (`get` on `configmaps`) in the `openshift-config-managed` namespace.
- If `jq` is not installed, install it via your package manager (`dnf install jq`, `apt install jq`, `brew install jq`, etc.).
- If no dashboards are returned, verify the label filter and that dashboards are present on your cluster.

---
