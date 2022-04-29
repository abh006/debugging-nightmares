# Deploying to K8s from GH Actions

This will be a lot explainatory. If you just want the gist, goto my GitHub Gists. Hopefully, you would find something there.

## Setup
I had :
  - a user `deploy` in hand for the GH actions purposes. Initially it had no access related to EKS
  - a `deploy.yaml` with the following step responsible for K8s deploy
    ```yaml
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.REGION_NAME }}
      - name: Apply Helmfile
        run: helmfile diff
    ```
    For debugging purposes I had changed the `helmfile apply` to `helmfile diff`
Github secrets were properly configured.

## Error
I was getting an error like "the cluster is unreachable. The server is asking for credentials"

At first I doubted the user `deploy` had enough permissions. Then I thought that wasn't the issue.
 

## Debug
When I did the preliminary debugging, (Google search, obviously), I found that the kubeconfig file was missing. 

I generated one (with the user `deploy` as my default user). Now, how to put it there?

I base64-encoded the contents of the file and added that string as a repo secret. `KUBECONFIG`
Then, accessed that from the GH actions and saved it to a file `~/.kube/config`
```yaml
- name: Configure Kubeconfig
  run: |
    mkdir $HOME/.kube 
    echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
    chmod 600 $HOME/.kube/config
    export KUBECONFIG=$HOME/.kube/config
```


Still the problem wasn't solved. Same error!
This time I went to the AWS IAM Console. I knew the permissions for the user `deploy` weren't enough. I added a new policy which contained `eks:DescribeClusters` and `eks:ListClusters` policies.
and re-ran the action once more.

Not the time for surprise. Same error!

After some more digging I found this [article](https://ianbelcher.me/tech-blog/setup-github-actions-for-eks-deployments). 

Actually, I was already half-way through this, when I found it.
I had to edit the config map and add my user `deploy` to it.
```yaml
apiVersion: v1
data:
  ...
  mapUsers: |
    ...
    - userarn: arn:aws:iam::<the-account-id>:user/<the-username>
      username: <the-username>
```
Then I created a ClusterRole and a ClusterRoleBinding as per the article. (Yes, copy paste).

cluster-role.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: github-action-eks-user-role
rules:
- apiGroups: ['*']
  resources: ["deployments","pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```
crb.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: github-action-eks-user-binding
subjects:
- kind: User
  name: <the-username>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: github-action-eks-user-role
  apiGroup: rbac.authorization.k8s.io
```
Vola, that step worked.

I knew that, when the error changed. Finally some progress!
```
User "deploy" cannot list resource "secrets" in API Group "" in the namespace "mynamespace"
```
Well, it looks like some permissions are still missing for the user `deploy`.
I went to the AWS IAM Console first, and checked if there is any permission related to this. But If you know, you know how foolish I was.

That permission was to be changed in the ClusterRole I just created. In the resources section "secrets" were missing
Like this:
```yaml
resources: ["deployments","pods","secrets"]
````

And that's where the action succeeds.
