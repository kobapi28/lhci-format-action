# lhci-format-action
GitHub Actions to format Lighthouse ci results for easy viewing.
Lighthouse ci and this action together with the action that comments on the pull request will comment on the pull request as follows.

[todo Place an image]

## Example
Incorporate the following code into your workflow.
Install and run lighthouse ci. Pass the file output there and the upload url to this action. The action will output a file called `result-markdown.md`, so set it to comment that file.

:warning:
As of v1.3, it will only work as described in `run Lighthouse ci`. We plan to fix this in the future.

```yml
      - name: install Lighthouse cli
        run: |
          yarn add @lhci/cli@0.8.x

      - name: run Lighthouse ci 
        id: lhci
        run: |
          mkdir out
          touch out/autorun.txt
          yarn lhci autorun | grep 'storage.googleapis.com' > ./out/autorun.txt
          report_url=$(cat ./out/autorun.txt)
          echo $report_url
          echo "::set-output name=report_url::$report_url"
        continue-on-error: true
        env: 
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}

      # this action!
      - name: json to markdown
        uses: shoma3571/lhci-format-action@v1.3
        id: parse
        with:
          json-file-path: .lighthouseci/assertion-results.json
          report-url: ${{ steps.lhci.outputs.report_url }}
      
      - name: post comment
        env: 
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          URL: ${{ github.event.pull_request.html_url }}
        run: gh pr comment -F ./result-markdown.md "${URL}"
```