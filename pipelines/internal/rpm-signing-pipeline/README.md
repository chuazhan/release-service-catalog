# rpm-signing-pipeline

Tekton pipeline for RPM signing. It is meant to be used by the rh-sign-rpm task, not as a standalone managed
pipeline.

## Parameters

| Name             | Description                                                                               | Optional | Default value                                                |
|------------------|-------------------------------------------------------------------------------------------|----------|--------------------------------------------------------------|
| pipeline_image   | An image with rh-signing-client needed for the signing                                    | No       | -                                                            |
| keytab_file      | Keytab file which is used when running with rh-signing-client tool                        | No       | -                                                            |
| secret_name      | Name of the Kubernetes Secret containing the keytab                                       | No       | -                                                            |
| sign_key_alias      | Sign key alias to use for signing                                                         | No       | -                                                            |
| component_name      | Name of the component to be signed                                                         | No       | -                                                            |
| component_artifact  | OCI artifact URL containing all component files (tarball)                                  | No       | -                                                            |
| ociStorage          | The OCI repository where the Trusted Artifacts are stored                                  | No       | -                                                            |
| requester        | Name of the user that requested the signing, for auditing purposes                        | No       | -                                                            |
| taskGitUrl       | The url to the git repo where the release-service-catalog tasks to be used are stored     | Yes      | https://github.com/konflux-ci/release-service-catalog.git    |
| taskGitRevision  | The revision in the taskGitUrl repo to be used                                            | No       | -                                                            |
