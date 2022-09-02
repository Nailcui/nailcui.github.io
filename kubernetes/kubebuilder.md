https://github.com/kubernetes-sigs/kubebuilder/releases

```shell
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
```



```
/usr/local/bin/kubebuilder_3.4.1 create webhook --group rollouts --version v1beta1 --kind Rollout --conversion
/usr/local/bin/kubebuilder_3.4.1 create webhook --group apps --version v1 --kind StatefulSet --conversion



```

