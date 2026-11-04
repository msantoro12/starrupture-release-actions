# starrupture-release-actions

Shared GitHub Actions for the StarRupture mod release workflows.

## nexus-publish

Uploads a file to a Nexus Mods page and, for a repo's primary upload, sets the
page Version field, adds the changelog entry and replaces the page description.
Uses `Nexus-Mods/upload-action@v1.0.0-beta.11`.

```yaml
- name: Upload Client build to Nexus Mods
  uses: msantoro12/starrupture-release-actions/nexus-publish@nexus-publish-v1
  with:
    api_key: ${{ secrets.NEXUS_APIKEY }}
    mod_id: '37074157699187'                      # global v3 id of the page
    file_id: ${{ secrets.NEXUS_CLIENT_GROUP_ID }}
    filename: ${{ github.workspace }}\Plugin-Client-${{ steps.tag.outputs.TAG }}.zip
    display_name: "Plugin Client ${{ steps.tag.outputs.TAG }}"
    version: ${{ steps.tag.outputs.NEXUS_VERSION }}
    file_description: "Changelog: https://github.com/${{ github.repository }}/releases/tag/${{ steps.tag.outputs.TAG }}"
    changelog: ${{ steps.assets.outputs.changelog }}
    description_file: nexus/description.bbcode    # optional
    tag: ${{ steps.tag.outputs.TAG }}
```

| Input | Notes |
|---|---|
| `api_key`, `mod_id`, `file_id`, `filename`, `display_name`, `version`, `file_description`, `changelog` | Required. `mod_id` is the global v3 id (`data.id` of `GET /v3/games/starrupture/mods/{page id}`). |
| `update_page` | `'true'` (default): primary upload, also sets the page Version, changelog and description. `'false'`: second upload (the Server file), file only; no changelog entry. |
| `description_file` | Optional BBCode file, relative to the workspace. The page description is replaced only when `update_page` is true and the file exists. |
| `tag`, `repo` | Substituted for `{{TAG}}` and `{{REPO}}` in the description file. `repo` defaults to the calling repository. A description that still has a `{{` afterwards is not sent. |

The description step uses an experimental Nexus endpoint, so a failure there emits a
`::warning::` and the job carries on. The upload step is not soft: a failure fails the job.
Changelog entries cannot be removed; never re-run a release for a version that already has its entry.

Callers pin the tag (`nexus-publish-v1`), not a branch.
