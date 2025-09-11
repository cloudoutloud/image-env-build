# image-env-build

To build and push generic images.

This repo is based on GCP but with changes could apply to any public cloud provider that has support with GitHub Actions.

Based on user input can build then push to following GCP projects registries (GAR).

- stg - stg-middleware-repo
- prod - prod-middleware-repo

Pipeline is triggered manually when needed with user inputted options.
Pipeline will fail if image if same tag is already exists in registry.
This is to ensure image are not overridden with same tag as the registry images are not set to immutable.

## How to add a image

1. Add a new folder under `images` name of folder can be anything this is not the same as image_name.

2. In folder add a image `Dockerfile` and `Makefile` example.

```
image_name:
	@echo "redis-commander"

tag:
	@echo "v1.0.0"
```

3. Define the specifci image_name you want to use and tag. This will be included in the full image path for example.

```
asia-northeast1-docker.pkg.dev/stg/stg-middleware-repo/github-runner:v1.0.0
```

4. Update `image_folder_name` user input list with the folder name so can be used as an option, example.

```
image_folder_name:
    description: 'Image folder name should match folder you added in PR'
    required: true
    type: choice
```
