---
title: "Upgrade Deployment"
metaTitle: "Upgrade Deployment"
metaDescription: "How to upgrade your instance to the latest OpenReplay version."
---

Upgrading your OpenReplay deployment to the latest version requires updating both your backend (instance) and tracker.

## Upgrade Backend (from v1.3.6 or prior)

First of all, ensure you're on `v1.3.6`. If that's already the case, then ignore the below commands and move to step 1). If not, update your OpenReplay installation to `v1.3.6`:
   
```bash 
# Copy/backup the current openreplay folder to _version-number
mv openreplay openreplay_v1.3.5
git clone https://github.com/openreplay/openreplay -b v1.3.6
cd openreplay/scripts/helm
# bash upgrade.sh <old openreplay path>
bash upgrade.sh ~/openreplay_v1.3.5
```

Once you're on `v1.3.6` then proceed with the below steps:

1. Copy/backup the current openreplay folder to `_version-number` like below:
   
  ```bash 
  mv openreplay openreplay_v1.3.6
  ```

2. Clone the new OpenReplay version. In this example we're upgrading to the latest available version:
   
  ```bash 
  git clone https://github.com/openreplay/openreplay
  ```

3. Run the below script to automatically upgrade your `vars.yaml` to the new format:
  
  ```bash
  cd openreplay/scripts/helmcharts
  ansible localhost -m template -a "src=vars_template.yaml dest=vars.yaml" -e @~/openreplay_v1.3.6/scripts/helm/vars.yaml
  ```

4. Upgrade OpenReplay:

  ```bash
  cd openreplay/scripts/helmcharts
  helm ls -n app | tail -n +2 | awk '{print $1}' | xargs -I{} helm uninstall {} -n app
  helm uninstall nginx-ingress -n nginx-ingress
  helm upgrade --install openreplay ./openreplay -n app -f vars.yaml --atomic --set forceMigration=true
  ```
  
5. If you're not using a load balancer and have generated your SSL certificate via the `certmanager.sh` script:

  ```
  cd openreplay/scripts/helmcharts && bash certmanager.sh
  ```

6. Update `fromVersion` variable in `/openreplay/scripts/helmcharts/vars.yaml` to reflect the new version. As an example if you're moving from `v1.3.6` to `v1.6.0` then update the `fromVersion` like below:
  
  ```yaml
  fromVersion: "v1.6.0"
  ```

> **Note:** 
Manual overrides made to any service configuration file (i.e. `openreplay/scripts/helm/app/<app>.yaml`) will be reset. So if you have any custom overrides, like using an [external object storage service](/configuration/recordings-storage) for your recordings, or increased service capacity (cpu/memory), make sure to apply them to the new version (in `/openreplay/scripts/helmcharts/vars.yaml`) prior to running the upgrade script (step 4).

## Upgrade Backend (from v1.4.0 or higher)

1. Copy/backup the current openreplay folder to `_version-number` like below:
   
  ```bash 
  mv openreplay openreplay_v1.4.0
  ```

2. Clone the new OpenReplay version. In this example we're upgrading to the latest available version:
   
  ```bash 
  git clone https://github.com/openreplay/openreplay
  ```

3. Upgrade OpenReplay:

  ```bash
  cd openreplay/scripts/helmcharts

  # Merge previous vars.yaml with current var.yaml (using yq for yaml parsing)
  cp ~/openreplay_v1.4.0/scripts/helmcharts/vars.yaml old_vars.yaml
  wget https://github.com/mikefarah/yq/releases/download/v4.24.4/yq_linux_amd64 -O yq
  chmod +x ./yq
  ./yq '. *= load("old_vars.yaml")' vars.yaml > new_vars.yaml
  mv new_vars.yaml vars.yaml
    
  # Upgrade openreplay
  helm upgrade --install openreplay ./openreplay -n app --wait -f ./vars.yaml --atomic
  ```
  
4. If you're not using a load balancer and have generated your SSL certificate via the `certmanager.sh` script:

  ```
  cd openreplay/scripts/helmcharts && bash certmanager.sh
  ```

5. Update `fromVersion` variable in `/openreplay/scripts/helmcharts/vars.yaml` to reflect the new version. As an example if you're moving from `v1.4.0` to `v1.6.0` then update the `fromVersion` like below:
  
  ```yaml
  fromVersion: "v1.6.0"
  ```
 
## Upgrade Tracker

Ensure your tracker (and tracker-assist plugin if you do use the Assist plugin) is compatible with the new backend version by checking the below compatibility table:

| Backend Version | Minimum Tracker Version | Minimum Assist Version (NPM) |
|----------|-------------|
| 1.6.0 | 3.5.10 | 3.5.8 |
| 1.5.4 | 3.5.4 | 3.5.7 |
| 1.5.3 | 3.5.3 | 3.5.5 |
| 1.5.2 | 3.5.2 | 3.5.4 |
| 1.5.1 | 3.5.1 | 3.5.3 |
| 1.5.0 | 3.5.0 | 3.5.0 |
| 1.4.0 | 3.4.17 | 3.4.16 |
| 1.3.6 | 3.4.16 | 3.4.15 |
| 1.3.5 | 3.4.0 | 3.4.13 |
| 1.3.0 | 3.2.1 | 3.4.12 |
| 1.2.0 | 3.1.0 | N/A |
| 1.1.0 | 3.0.3 | N/A |
