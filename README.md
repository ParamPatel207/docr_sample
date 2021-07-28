Deploy a [DigitalOcean App Platform](https://www.digitalocean.com/products/app-platform/) app using GitHub Actions.

 - Auto-deploy your app from source on commit, while allowing you to run tests or perform other operations this action also allows you to update docr images in the [DigitalOcean App Platform App Spec](https://docs.digitalocean.com/products/app-platform/references/app-specification-reference/).

## Example

 Add your `.do/app.yaml`:

**Note that you should not configure `deploy_on_push: true` for this workflow.**

```yaml
name: sample-dockerfile
services:
- dockerfile_path: Dockerfile
  github:
    branch: main
    deploy_on_push: true
    repo: digitalocean/sample-dockerfile
  name: sample-dockerfile
```

Create a `.github/workflows/main.yml`:
- Push docker images to the DigitalOcean Container Registry by following the steps shown below:
**Note: Below example uses github_sha as the image tag which will help for instant deployment to DigitalOcean App Platform.**
```yaml
- name: Install doctl
      uses: digitalocean/action-doctl@v2
      with:
        token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
    - name: Generate GITHUB_SHA
      id: github-sha
      shell: bash
      run: |
        SHORT_SHA=$(echo $GITHUB_SHA | cut -c1-7)
        echo "::set-output name=sha::$SHORT_SHA"
    - name: Publish Image to Digital Ocean Container Registry
      shell: bash
      run: |
        doctl registry login --expiry-seconds 600
        docker build . -t registry.digitalocean.com/sample-go/add_sample:${{steps.github-sha.outputs.sha}}
        docker push registry.digitalocean.com/sample-go/add_sample:${{steps.github-sha.outputs.sha}}
```
**Note: Always use unique tag names to push image to the DigitalOcean Container Registry. This will allow you to deploy your application without delay. [ref](https://docs.digitalocean.com/products/container-registry/quickstart/)**

## How its using DigitalOcean App Platform App Actions?

This example is using [DigitalOcean App Platform App Actions](https://github.com/ParamPatel207/app_action) in the .github/workflows/main.yml file to auto-deploy your app.

```yaml
 - name: DigitalOcean App Platform deployment
      uses: ParamPatel207/app_action@main
      with:
        app_name: sample-golang
        token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
        list_of_image: '[
                          {
                            "name": "web",
                            "repository": "registry.digitalocean.com/sample-go/add_sample",
                            "tag": "${{steps.github-sha.outputs.sha}}"
                          }
                        ]'

```
## Note for handling DigitalOcean Container Registry images: 
Because image manifests are cached in different regions, there may be a maximum delay of one hour between pushing to a tag that already exists in your registry and being able to pull the new image by tag. This may happen, for example, when using the :latest tag. To avoid the delay, use:

- Unique tags (other than :latest)
- SHA hash of Github commit
- SHA hash of the new manifest