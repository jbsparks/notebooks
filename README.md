# notebooks
Jupiter notebooks

## Convert helm/tiller to kubectl

```bash
$ mkdir charts manifests
$ helm fetch --untar --untardir ./charts jupyterhub/jupyterhub
$ cd charts
$ helm template --set proxy.secretToken="b326bae33a6793ab57c5ca3677c540c4d53a4be193c7551adcb0778018e05d97" jupyterhub --output-dir ~/manifests
$ cd ..
$ kubectl apply --recursive --filename ~/manifests/jupyterhub
```
