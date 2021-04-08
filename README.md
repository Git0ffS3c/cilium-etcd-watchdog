# cilium-etcd-watchdog

A watchdog for Cilium's etcd.

The purpose of this watchdog is to delete ETCD Cluster k8s resources when the quorum is lost or of unknown state for significant amount of time.
It was created because [cilium-etcd-operator](https://github.com/cilium/cilium-etcd-operator) at the time did not tried to handle quorum lost because of state correctness concerns which in our use case are not relevant.

It is expected that other subjects will create the cluster once its deleted, for example Flux.

## Algorithm details

Watchdog algorithm sits in endless loop that triggers in time intervals (controlled via `polling-interval` flag, default `10 sec`).

It loops waiting for cilium ETCD cluster to be become available for maximum of bootstrap grace period (controlled via `cluster-bootstrap-grace-period` flag, default `2 min`).

After witch it will check ETCD quorum status.
If quorum is lost it will delete ETCD Cluster k8s resources.
If the quorum state is unknown the loop will be repeated for number of times (controlled by `max-quorum-status-check-failures` flag, default `3`) after which if it still in the unknown state will result in delete of ETCD Cluster k8s resources.
