---
title: "Tidbits: Ceph RGW - List All S3 Buckets Disk Usage"
# description: ""
date: "2025-05-11T13:23:18+08:00"
lastmod: "2025-05-11T13:23:18+08:00"
slug: "2025/ceph-rgw-list-all-s3-buckets-disk-usage"
toc: true
categories: [Tidbits, Kubernetes, Storage]
---

Quick one liner command to list all Ceph RGW buckets usage, using `jq` to extract only the bucket name and size KV pairs and converting from bytes to gigabytes (GB):

```sh
radosgw-admin bucket stats | jq 'def calc(val): if val!=null then val|tonumber/1000000000|tostring + "GB" else . end; .[] | {bucket} + (.usage."rgw.main" | {size: (calc(.size)), size_actual: (calc(.size_actual)), size_utilized: (calc(.size_utilized))})'
```

For me, I use Rook-Ceph on Kubernetes for my homelab, and like to view the output in Neovim, so the full command line for me looks like this:

```sh
kubectl exec -it -n rook-ceph deploy/rook-ceph-tools -- radosgw-admin bucket stats | jq 'def calc(val): if val!=null then val|tonumber/1000000000|tostring + "GB" else . end; .[] | {bucket} + (.usage."rgw.main" | {size: (calc(.size)), size_actual: (calc(.size_actual)), size_utilized: (calc(.size_utilized))})' | nvim
```

Or only the `jq` query if you prefer:

```json
def calc(val): if val!=null then val|tonumber/1000000000|tostring + "GB" else . end;

.[] | {bucket} + (.usage."rgw.main" | {
    size: (calc(.size)),
    size_actual: (calc(.size_actual)), 
    size_utilized: (calc(.size_utilized))
})
```

Here's an example output:

```json
{
  "bucket": "pg-authentik",
  "size": "1.688277968GB",
  "size_actual": "1.725648896GB",
  "size_utilized": "1.688277968GB"
}
{
  "bucket": "pg-gts-robo",
  "size": "0.003353182GB",
  "size_actual": "0.003444736GB",
  "size_utilized": "0.003353182GB"
}
{
  "bucket": "joplin",
  "size": null,
  "size_actual": null,
  "size_utilized": null
}
{
  "bucket": "pg-atuin",
  "size": "0.468781088GB",
  "size_actual": "0.48617472GB",
  "size_utilized": "0.468781088GB"
}
{
  "bucket": "gotosocial-media",
  "size": "7.746584026GB",
  "size_actual": "7.825215488GB",
  "size_utilized": "7.746584026GB"
}
{
  "bucket": "gts-robo-media",
  "size": null,
  "size_actual": null,
  "size_utilized": null
}
{
  "bucket": "zipline-data",
  "size": "0.00079149GB",
  "size_actual": "0.000794624GB",
  "size_utilized": "0.00079149GB"
}
{
  "bucket": "pg-home",
  "size": "2.763082224GB",
  "size_actual": "2.787741696GB",
  "size_utilized": "2.763082224GB"
}
{
  "bucket": "reactive-resume-media",
  "size": null,
  "size_actual": null,
  "size_utilized": null
}
{
  "bucket": "pg-gotosocial-valetudo",
  "size": null,
  "size_actual": null,
  "size_utilized": null
}
{
  "bucket": "volsync",
  "size": "373.431380366GB",
  "size_actual": "373.502193664GB",
  "size_utilized": "373.431380366GB"
}
{
  "bucket": "pg-default",
  "size": "2.899299768GB",
  "size_actual": "2.965950464GB",
  "size_utilized": "2.899299768GB"
}
{
  "bucket": "k8s-schemas-rgw",
  "size": "0.0391532GB",
  "size_actual": "0.040087552GB",
  "size_utilized": "0.0391532GB"
}
{
  "bucket": "pg-gotosocial",
  "size": "6.635240072GB",
  "size_actual": "6.69478912GB",
  "size_utilized": "6.635240072GB"
}
```
