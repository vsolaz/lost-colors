# The Legend of Lost Colors

In this GameDay you build a multimodal investigator agent with Amazon Bedrock
AgentCore. The agent uses tools for media analysis, locations and flight data,
connected through an AgentCore Gateway, to solve a series of crystal heists.

## How the GameDay works

The **GameDay console** in Workshop Studio runs the game. It tells the story,
gives you each task and its subtasks, and checks your answers. Each correct
answer unlocks the next step of the quest, so treat the console as your main
guide.

This **readme** is your field manual. It walks you through the initial setup
(tools, credentials, Python packages, IAM role and Cognito) and then gives you
the commands, settings and hints you need for each task.

The usual flow:

1. **Start here in the readme** and complete Steps 1 to 5. The GameDay tasks
   assume this setup is done.
2. **Read the current task in the GameDay console.**
3. **Come back to the readme step for that task** (see the table below), follow
   it, and submit your answer in the console.
4. Repeat for the next task. You switch back and forth until all three tasks
   are complete.

Follow the readme steps in order. Unless a step says otherwise, run every
command from the `lost-colors` directory (the folder that contains this
`readme.md`).

## Where you work

You work in three places:

- **Your terminal**, where you run the commands in this readme.
- **The GameDay console** in Workshop Studio, where you read each task and
  submit your answers.
- **The AWS console**, opened from Workshop Studio, for the steps you do in the
  browser.

Boxes that start with **Switch to the GameDay console** or **Switch to the AWS
console** tell you when to change windows and what to do there.

**Tip: copy commands with one click.** Install the Markdown Code Copy Button
extension in Kiro by running this in your terminal:

```bash
kiro --install-extension barnim.markdown-code-copy-button
```

If your terminal says `kiro: command not found`, install it from Kiro instead:
open the Extensions view (`Cmd+Shift+X`, Windows/Linux: `Ctrl+Shift+X`), search
for `barnim.markdown-code-copy-button` and click **Install**.

Then open this readme in preview with `Cmd+Shift+V` (Windows/Linux:
`Ctrl+Shift+V`). Every command box gets a copy button in its top-right corner.
Copying instead of typing avoids most mistakes.

| GameDay task | Readme step | What you submit |
|---|---|---|
| Task 1, Subtask A: AgentCore Runtime | Step 8 | The runtime ID of `multimodal_data_agent` |
| Task 1, Subtask B: AgentCore Identity | Step 6 | A credential provider name, for example `agent-to-gateway-token` |
| Task 1, Subtask C: AgentCore Gateway | Step 9 | The Gateway ID |
| All of Task 1 | Step 10 | Nothing; you complete the agent code's `<TODO>`s and redeploy |
| Task 2, Subtasks A, B and C | Step 11 | The target ID of the `bda`, `location` and `flights` targets |
| Task 3: Investigations | Step 12 | The answers to the seven questions |

The readme creates the identities (Subtask B) before it deploys the runtime
(Subtask A), because the deployment needs the Cognito values. If the console
only accepts the subtasks in order, keep the Step 6 answer and submit it right
after the Step 8 one.

> **Windows users:** the commands below are written for macOS and Linux
> terminals. Where a command is different on Windows, the PowerShell version is
> shown next to it.

## Step 1: Install the required tools

You need these tools on your computer. If you already have Python and Node.js,
you can keep them, but install the latest AWS CLI in every case. Then run the
version check at the end of this step.

| Tool | Minimum version | Used for | Download |
|---|---|---|---|
| Python | 3.10 | Setup scripts and the agent code | <https://www.python.org/downloads/> |
| Node.js | 20.12 | Runs the AgentCore CLI | <https://nodejs.org/> (pick the LTS version) |
| AWS CLI | Latest version (see below) | Talking to your AWS account | <https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html> |

You do not need Docker. The agent containers are built in your AWS account
during deployment.

### Install the latest AWS CLI (required, even if you already have it)

The quest uses `aws bedrock-agentcore-control` commands, which older AWS CLI
versions do not have. Install the latest version now, even if `aws` already
works on your computer. Installing over an existing AWS CLI updates it and keeps
your configuration.

macOS:

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

Linux (x86_64; on ARM use `awscli-exe-linux-aarch64.zip`):

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install --update
```

Windows (PowerShell):

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

If you installed the AWS CLI with Homebrew, pip or another package manager,
use the commands above anyway, then run `type -a aws` (Windows: `where.exe aws`)
and make sure the first result is the version you just installed.

### Install the AgentCore CLI

After installing Node.js, install the AgentCore CLI:

```bash
npm install -g @aws/agentcore@1.0.1
```

If you get a permission error (`EACCES`) on macOS or Linux, do not use `sudo`.
Install Node.js with [nvm](https://github.com/nvm-sh/nvm) instead and run the
command again.

Close and reopen your terminal, then check that everything is installed:

```bash
python3 --version     # Windows: python --version
node --version        # must be v20.12 or newer
aws --version
aws bedrock-agentcore-control help > /dev/null && echo "AWS CLI OK"   # Windows: aws bedrock-agentcore-control help
agentcore --version   # must print 1.0.1
```

If the AWS CLI check prints `Invalid choice` instead of `AWS CLI OK`, your
terminal is still using an old AWS CLI. Close and reopen the terminal; if it
still fails, run the install commands above again.

If `agentcore --version` reports `No such option: --version`, your terminal is
finding the old Python version of the AgentCore CLI. Remove only the old
starter toolkit and refresh the terminal's command cache:

```bash
python3 -m pip uninstall bedrock-agentcore-starter-toolkit
hash -r
type -a agentcore
agentcore --version
```

Do not uninstall `bedrock-agentcore`; the multimodal agent uses that Python package at runtime.

## Step 2: Get your AWS credentials from Workshop Studio

Every team gets its own AWS account for the GameDay. Your terminal needs
temporary credentials for that account.

1. Open the GameDay event page in Workshop Studio.
2. Click **Get AWS CLI credentials** (in the **AWS account access** section).
3. Pick the tab that matches your terminal: **Mac/Linux**, **Windows PowerShell**,
   or **Windows cmd**.
4. Copy the whole block. It contains `AWS_DEFAULT_REGION`, `AWS_ACCESS_KEY_ID`,
   `AWS_SECRET_ACCESS_KEY` and `AWS_SESSION_TOKEN`.
5. Paste it into your terminal and press Enter.

Check that the terminal is now using the GameDay account:

```bash
aws sts get-caller-identity
```

The `Account` value must match the account ID shown in Workshop Studio. If it
shows a different account (for example, your own personal or university
account), paste the Workshop Studio credentials again in this terminal.

Keep these rules in mind for the rest of the GameDay:

- **Credentials only apply to the terminal window you pasted them into.** If you
  open a new terminal or tab, paste them again.
- **Credentials expire.** If a command fails with `ExpiredToken`,
  `InvalidClientTokenId` or `Unable to locate credentials`, go back to Workshop
  Studio, click **Get AWS CLI credentials** again and paste the new block.
- **Never commit credentials** to Git or share them in chat.

## Step 3: Install the Python dependencies

### Check the MCP library version

The agent and the two MCP tool servers are written for version 1 of the `mcp`
Python library. Version 2 renamed some of the classes they use, so if a runtime
gets version 2, it crashes at startup with `ImportError: cannot import name
'streamablehttp_client'` or `No module named 'mcp.server.fastmcp'`. The
runtimes' `requirements.txt` files already limit `mcp` to version 1. Check that
each of the three files has exactly one `mcp` line ending in `<2`:

```bash
grep -H "^mcp" agent/requirements.txt tools/*/requirements.txt
# Windows PowerShell: Select-String "^mcp" agent\requirements.txt, tools\*\requirements.txt
```

These files are installed inside the runtime containers during
`agentcore deploy` (Step 8). Do not change the `mcp` lines.

### Create the virtual environment

Create a Python virtual environment (an isolated folder for this project's
Python packages) and install the packages into it:

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows PowerShell: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Your prompt now starts with `(.venv)`. If you open a new terminal later, run the
`activate` command again (and paste your credentials again).

## Step 4: Create the IAM role

All AgentCore resources in this quest use one IAM role called `LostColors`. It
gives the agent and tools permission to call other AWS services. Create it
before doing anything else:

```bash
python3 IAM-setup.py
```

If the script says the role already exists, that is fine. A teammate already
created it.

## Step 5: Create the Cognito login pools and load your quest variables

The tools only accept requests that carry a valid token. These tokens come from
Amazon Cognito. Run:

```bash
python3 identity/identity_setup.py
```

The script creates two Cognito user pools and prints a block of details for
each:

| Output block | Used for |
|---|---|
| `IDP details for agent-to-gateway ->` | The agent logging in to the AgentCore Gateway (Step 9) |
| `IDP details for gateway-to-runtime ->` | The Gateway logging in to the MCP tool runtimes (Step 7) |

Copy the full output into a text file. It contains **client secrets**, which you
need later in the quest. Keep the file private and do not commit it. If you
lose the output, run the script again; it finds the existing pools and prints
the same values.

### Load your quest variables

Instead of copying IDs by hand, look them up with the AWS CLI and save them as
variables in your terminal. Copy the whole block for your terminal, paste it,
and press Enter.

macOS / Linux:

```bash
export AWS_REGION="${AWS_DEFAULT_REGION:-$(aws configure get region)}"
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export LOSTCOLORS_ROLE_ARN=$(aws iam get-role --role-name LostColors --query Role.Arn --output text)
export GATEWAY_USER_POOL_ID=$(aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='agent-to-gateway-pool'].Id | [0]" --output text)
export GATEWAY_CLIENT_ID=$(aws cognito-idp list-user-pool-clients --user-pool-id "$GATEWAY_USER_POOL_ID" --query "UserPoolClients[?ClientName=='agent-to-gateway-client'].ClientId | [0]" --output text)
export RUNTIME_USER_POOL_ID=$(aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='gateway-to-runtime-pool'].Id | [0]" --output text)
export RUNTIME_CLIENT_ID=$(aws cognito-idp list-user-pool-clients --user-pool-id "$RUNTIME_USER_POOL_ID" --query "UserPoolClients[?ClientName=='gateway-to-runtime-client'].ClientId | [0]" --output text)
export BDA_BUCKET_NAME=$(aws s3api list-buckets --query "Buckets[?contains(Name, 'intelligencebucket')].Name | [0]" --output text)
env | grep -E '^(AWS_REGION|ACCOUNT_ID|LOSTCOLORS_ROLE_ARN|GATEWAY_USER_POOL_ID|GATEWAY_CLIENT_ID|RUNTIME_USER_POOL_ID|RUNTIME_CLIENT_ID|BDA_BUCKET_NAME)='
```

Windows PowerShell:

```powershell
$env:AWS_REGION = if ($env:AWS_DEFAULT_REGION) { $env:AWS_DEFAULT_REGION } else { aws configure get region }
$env:ACCOUNT_ID = aws sts get-caller-identity --query Account --output text
$env:LOSTCOLORS_ROLE_ARN = aws iam get-role --role-name LostColors --query Role.Arn --output text
$env:GATEWAY_USER_POOL_ID = aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='agent-to-gateway-pool'].Id | [0]" --output text
$env:GATEWAY_CLIENT_ID = aws cognito-idp list-user-pool-clients --user-pool-id $env:GATEWAY_USER_POOL_ID --query "UserPoolClients[?ClientName=='agent-to-gateway-client'].ClientId | [0]" --output text
$env:RUNTIME_USER_POOL_ID = aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='gateway-to-runtime-pool'].Id | [0]" --output text
$env:RUNTIME_CLIENT_ID = aws cognito-idp list-user-pool-clients --user-pool-id $env:RUNTIME_USER_POOL_ID --query "UserPoolClients[?ClientName=='gateway-to-runtime-client'].ClientId | [0]" --output text
$env:BDA_BUCKET_NAME = aws s3api list-buckets --query "Buckets[?contains(Name, 'intelligencebucket')].Name | [0]" --output text
Get-ChildItem env:AWS_REGION, env:ACCOUNT_ID, env:LOSTCOLORS_ROLE_ARN, env:GATEWAY_USER_POOL_ID, env:GATEWAY_CLIENT_ID, env:RUNTIME_USER_POOL_ID, env:RUNTIME_CLIENT_ID, env:BDA_BUCKET_NAME
```

The last line prints every variable. First check that `ACCOUNT_ID` matches the
account ID shown in Workshop Studio. If it does not, your terminal is using
other credentials: paste the Workshop Studio credentials (Step 2) and run the
block again. Then check that each variable has a real value. If a
value is empty or `None`, or you see an error, the most common causes are
expired credentials (Step 2) or a skipped step (Step 4 or the
`identity_setup.py` command above). Fix the cause and paste the block again.

| Variable | Contains |
|---|---|
| `AWS_REGION` | Your AWS Region, for example `us-east-1` |
| `ACCOUNT_ID` | Your 12-digit AWS account ID |
| `LOSTCOLORS_ROLE_ARN` | ARN of the `LostColors` IAM role |
| `GATEWAY_USER_POOL_ID`, `GATEWAY_CLIENT_ID` | The `agent-to-gateway` Cognito user pool and client |
| `RUNTIME_USER_POOL_ID`, `RUNTIME_CLIENT_ID` | The `gateway-to-runtime` Cognito user pool and client |
| `BDA_BUCKET_NAME` | The S3 bucket the Bedrock Data Automation tool stores media in |

To use a variable in a command, write `"$NAME"` on macOS/Linux or `$env:NAME`
in PowerShell. To see a value, run `echo "$NAME"` (PowerShell: `$env:NAME`).

These commands do not look up client secrets. Take those from the
`identity_setup.py` output when you need them.

> **Every time you open a new terminal**, do these things in it before
> anything else:
>
> 1. Paste your Workshop Studio credentials (Step 2).
> 2. `cd` into the `lost-colors` directory.
> 3. Activate the virtual environment (Step 3).
> 4. Paste the variables block above.
> 5. If you already deployed, also paste the runtime variables block from Step 8.
> 6. If you already created the Gateway, also paste the Gateway variables block
>    from Step 9.

## Step 6: Create the AgentCore Identity credential providers

The agent and the Gateway do not keep Cognito client secrets in their code.
Instead, AgentCore Identity stores them in **OAuth2 credential providers**, and
the code asks AgentCore Identity for a token by the provider's name. You need
two providers, one for each Cognito pool from Step 5:

| Credential provider name | Cognito pool | Used for |
|---|---|---|
| `agent-to-gateway-token` | `agent-to-gateway-pool` | The agent getting a token to call the AgentCore Gateway |
| `gateway-to-runtime-token` | `gateway-to-runtime-pool` | The Gateway getting a token to call the MCP tool runtimes |

Make sure the Step 5 variables are set in this terminal, then paste the block
for your terminal. It reads each client secret directly from Cognito, so you do
not have to copy secrets by hand. It is safe to run more than once; existing
providers are left unchanged.

macOS / Linux:

```bash
python3 - <<'EOF'
import os, boto3
region = os.environ.get("AWS_REGION", "")
providers = {
    "agent-to-gateway-token": ("GATEWAY_USER_POOL_ID", "GATEWAY_CLIENT_ID"),
    "gateway-to-runtime-token": ("RUNTIME_USER_POOL_ID", "RUNTIME_CLIENT_ID"),
}
cognito = boto3.client("cognito-idp", region_name=region or None)
identity = boto3.client("bedrock-agentcore-control", region_name=region or None)
for name, (pool_var, client_var) in providers.items():
    pool_id, client_id = os.environ.get(pool_var, ""), os.environ.get(client_var, "")
    if "None" in (region, pool_id, client_id) or "" in (region, pool_id, client_id):
        print(f"{name}: skipped, AWS_REGION, {pool_var} or {client_var} is not set. Paste the variables block from Step 5 again.")
        continue
    try:
        existing = identity.get_oauth2_credential_provider(name=name)
        print(f"{name}: already exists ({existing['credentialProviderArn']})")
        continue
    except identity.exceptions.ResourceNotFoundException:
        pass
    secret = cognito.describe_user_pool_client(UserPoolId=pool_id, ClientId=client_id)["UserPoolClient"]["ClientSecret"]
    created = identity.create_oauth2_credential_provider(
        name=name,
        credentialProviderVendor="CustomOauth2",
        oauth2ProviderConfigInput={"customOauth2ProviderConfig": {
            "oauthDiscovery": {"discoveryUrl": f"https://cognito-idp.{region}.amazonaws.com/{pool_id}/.well-known/openid-configuration"},
            "clientId": client_id,
            "clientSecret": secret,
        }},
    )
    print(f"{name}: created ({created['credentialProviderArn']})")
EOF
```

Windows PowerShell:

```powershell
@'
import os, boto3
region = os.environ.get("AWS_REGION", "")
providers = {
    "agent-to-gateway-token": ("GATEWAY_USER_POOL_ID", "GATEWAY_CLIENT_ID"),
    "gateway-to-runtime-token": ("RUNTIME_USER_POOL_ID", "RUNTIME_CLIENT_ID"),
}
cognito = boto3.client("cognito-idp", region_name=region or None)
identity = boto3.client("bedrock-agentcore-control", region_name=region or None)
for name, (pool_var, client_var) in providers.items():
    pool_id, client_id = os.environ.get(pool_var, ""), os.environ.get(client_var, "")
    if "None" in (region, pool_id, client_id) or "" in (region, pool_id, client_id):
        print(f"{name}: skipped, AWS_REGION, {pool_var} or {client_var} is not set. Paste the variables block from Step 5 again.")
        continue
    try:
        existing = identity.get_oauth2_credential_provider(name=name)
        print(f"{name}: already exists ({existing['credentialProviderArn']})")
        continue
    except identity.exceptions.ResourceNotFoundException:
        pass
    secret = cognito.describe_user_pool_client(UserPoolId=pool_id, ClientId=client_id)["UserPoolClient"]["ClientSecret"]
    created = identity.create_oauth2_credential_provider(
        name=name,
        credentialProviderVendor="CustomOauth2",
        oauth2ProviderConfigInput={"customOauth2ProviderConfig": {
            "oauthDiscovery": {"discoveryUrl": f"https://cognito-idp.{region}.amazonaws.com/{pool_id}/.well-known/openid-configuration"},
            "clientId": client_id,
            "clientSecret": secret,
        }},
    )
    print(f"{name}: created ({created['credentialProviderArn']})")
'@ | python -
```

Each provider prints `created` or `already exists`, followed by its ARN. Check
that both are listed:

```bash
aws bedrock-agentcore-control list-oauth2-credential-providers --query "credentialProviders[].name" --output text
```

Use these names wherever the quest asks for a credential provider. The agent
code uses `agent-to-gateway-token` (Step 10), and the BDA and Location Gateway
targets use `gateway-to-runtime-token` (Step 11).

> **Switch to the GameDay console (Task 1, Subtask B).** Submit
> `agent-to-gateway-token` as your credential provider name. The subtask also
> asks you to put this name in the agent code; you do that in Step 10.

## Step 7: Fill in the AgentCore project configuration

The file `agentcore/agentcore.json` describes everything that gets deployed. It
contains placeholders that start with `YOUR_`. Each one is replaced with one of
your variables:

| Placeholder | Replaced with |
|---|---|
| `YOUR_ACCOUNT_ID` | `ACCOUNT_ID` |
| `YOUR_REGION` | `AWS_REGION` |
| `YOUR_RUNTIME_USER_POOL_ID` | `RUNTIME_USER_POOL_ID` (the `gateway-to-runtime` pool) |
| `YOUR_RUNTIME_CLIENT_ID` | `RUNTIME_CLIENT_ID` (the `gateway-to-runtime` client) |
| `YOUR_BDA_BUCKET_NAME` | `BDA_BUCKET_NAME` |

Paste the block for your terminal to fill them all in. It changes nothing if a
variable is missing.

macOS / Linux:

```bash
python3 - <<'EOF'
import json, os
path = "agentcore/agentcore.json"
values = {
    "YOUR_ACCOUNT_ID": "ACCOUNT_ID",
    "YOUR_REGION": "AWS_REGION",
    "YOUR_RUNTIME_USER_POOL_ID": "RUNTIME_USER_POOL_ID",
    "YOUR_RUNTIME_CLIENT_ID": "RUNTIME_CLIENT_ID",
    "YOUR_BDA_BUCKET_NAME": "BDA_BUCKET_NAME",
}
missing = [v for v in values.values() if os.environ.get(v, "None") in ("", "None")]
if missing:
    print("Not set:", ", ".join(missing), "- paste the variables block from Step 5 again.")
else:
    text = open(path).read()
    for placeholder, variable in values.items():
        text = text.replace(placeholder, os.environ[variable])
    json.loads(text)
    open(path, "w").write(text)
    print("Updated", path)
EOF
```

Windows PowerShell:

```powershell
$path = "agentcore\agentcore.json"
$values = [ordered]@{
    YOUR_ACCOUNT_ID           = $env:ACCOUNT_ID
    YOUR_REGION               = $env:AWS_REGION
    YOUR_RUNTIME_USER_POOL_ID = $env:RUNTIME_USER_POOL_ID
    YOUR_RUNTIME_CLIENT_ID    = $env:RUNTIME_CLIENT_ID
    YOUR_BDA_BUCKET_NAME      = $env:BDA_BUCKET_NAME
}
$missing = @($values.Keys | Where-Object { -not $values[$_] -or $values[$_] -eq "None" })
if ($missing) {
    Write-Host "No value for: $($missing -join ', ') - paste the variables block from Step 5 again."
} else {
    $text = Get-Content $path -Raw
    foreach ($placeholder in $values.Keys) { $text = $text.Replace($placeholder, $values[$placeholder]) }
    Set-Content $path $text -NoNewline
    Write-Host "Updated $path"
}
```

You can also open `agentcore/agentcore.json` and replace the placeholders by
hand with the values the variables block printed. If you do, use the
**`gateway-to-runtime`** values, not the `agent-to-gateway` ones.

Check that no placeholders are left. This command must print nothing:

```bash
grep YOUR_ agentcore/agentcore.json     # Windows PowerShell: Select-String YOUR_ agentcore\agentcore.json
```

The project deploys three runtimes together:

- `multimodal_data_agent`: the agent itself, called over HTTP.
- `bedrock_data_automation_mcp`: an MCP tool server for media analysis, protected by Cognito tokens (custom JWT authorization).
- `location_service_mcp`: an MCP tool server for location data, protected by Cognito tokens (custom JWT authorization).

## Step 8: Deploy

Check again that your terminal uses the GameDay account
(`aws sts get-caller-identity`), then run from the `lost-colors` directory:

```bash
agentcore deploy --yes
```

`agentcore deploy` checks the project before deploying. The first deployment
takes about 5 minutes because it also prepares the account for deployments.
It deploys into whichever AWS account and Region your current credentials
point to, which is why the account check above matters.

Whenever you change code or `agentcore/agentcore.json`, run `agentcore deploy`
again.

### Load the runtime variables

When the deployment finishes, save the runtime IDs and ARNs as variables.
Paste the block for your terminal:

macOS / Linux:

```bash
export AGENT_RUNTIME_ID=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_multimodal_data_agent'].agentRuntimeId | [0]" --output text)
export BDA_RUNTIME_ID=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_bedrock_data_automation_mcp'].agentRuntimeId | [0]" --output text)
export LOCATION_RUNTIME_ID=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_location_service_mcp'].agentRuntimeId | [0]" --output text)
export AGENT_RUNTIME_ARN=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_multimodal_data_agent'].agentRuntimeArn | [0]" --output text)
export BDA_RUNTIME_ARN=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_bedrock_data_automation_mcp'].agentRuntimeArn | [0]" --output text)
export LOCATION_RUNTIME_ARN=$(aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_location_service_mcp'].agentRuntimeArn | [0]" --output text)
env | grep -E '^(AGENT|BDA|LOCATION)_RUNTIME_(ID|ARN)='
```

Windows PowerShell:

```powershell
$env:AGENT_RUNTIME_ID = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_multimodal_data_agent'].agentRuntimeId | [0]" --output text
$env:BDA_RUNTIME_ID = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_bedrock_data_automation_mcp'].agentRuntimeId | [0]" --output text
$env:LOCATION_RUNTIME_ID = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_location_service_mcp'].agentRuntimeId | [0]" --output text
$env:AGENT_RUNTIME_ARN = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_multimodal_data_agent'].agentRuntimeArn | [0]" --output text
$env:BDA_RUNTIME_ARN = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_bedrock_data_automation_mcp'].agentRuntimeArn | [0]" --output text
$env:LOCATION_RUNTIME_ARN = aws bedrock-agentcore-control list-agent-runtimes --query "agentRuntimes[?agentRuntimeName=='LostColors_location_service_mcp'].agentRuntimeArn | [0]" --output text
Get-ChildItem env:AGENT_RUNTIME_ID, env:AGENT_RUNTIME_ARN, env:BDA_RUNTIME_ID, env:BDA_RUNTIME_ARN, env:LOCATION_RUNTIME_ID, env:LOCATION_RUNTIME_ARN
```

Each runtime ID looks like `LostColors_multimodal_data_agent-AbCdEf1234`, and
each ARN must contain your GameDay account ID. If a value is `None`, the
deployment has not finished, failed, or the terminal is using the wrong
credentials. Check `aws sts get-caller-identity` and the `agentcore deploy`
output, then paste the block again.

> **Switch to the GameDay console (Task 1, Subtask A).** You can submit your
> first answer now. In the GameDay console, open the **The Legend of Lost
> Colors** quest and go to Task 1, Subtask A. Submit the value of
> `AGENT_RUNTIME_ID` (`echo "$AGENT_RUNTIME_ID"`; PowerShell:
> `$env:AGENT_RUNTIME_ID`). It looks like
> `LostColors_multimodal_data_agent-AbCdEf1234`. Submit the ID, not the ARN.
> If the answer is not accepted, wait until
> `agentcore runtime get --id "$AGENT_RUNTIME_ID" --json` shows
> `"status": "READY"` and submit again. Then come back here and continue.

| Variable | Contains |
|---|---|
| `AGENT_RUNTIME_ID`, `AGENT_RUNTIME_ARN` | The `multimodal_data_agent` runtime |
| `BDA_RUNTIME_ID`, `BDA_RUNTIME_ARN` | The `bedrock_data_automation_mcp` runtime |
| `LOCATION_RUNTIME_ID`, `LOCATION_RUNTIME_ARN` | The `location_service_mcp` runtime |

To see all three runtimes and their deployment state:

```bash
agentcore status --json
```

You may see `This is an AgentCore CLI v0 project running in v1 compatibility
mode`. This message is expected and you can ignore it.

To inspect one runtime:

```bash
agentcore runtime get --id "$AGENT_RUNTIME_ID" --json
# Windows PowerShell: agentcore runtime get --id $env:AGENT_RUNTIME_ID --json
```

Use `"$BDA_RUNTIME_ID"` or `"$LOCATION_RUNTIME_ID"` for the tool runtimes.

Test the agent runtime:

```bash
agentcore invoke --runtime multimodal_data_agent --payload '{"prompt": "Hello"}'
```

**At this point an error is expected.** The agent code in
`agent/multimodal_data_agent.py` still contains `<TODO>` placeholders that you
complete in Step 10: the credential provider name, the AgentCore Gateway URL
and the agent instructions. Until then, the invoke returns something like:

```text
data: {"error": "An error occurred (ValidationException) when calling the GetResourceOauth2Token operation: ... 'resourceCredentialProviderName' failed to satisfy constraint ..."}
```

This error means the deployment worked: the runtime started and ran your
request. If you instead get `An error occurred when starting the runtime`, the
runtime did not start; see Troubleshooting.


## Step 9: Create the AgentCore Gateway

The AgentCore Gateway puts all the investigation tools behind one MCP endpoint.
The agent logs in to it with a token from the **`agent-to-gateway`** Cognito
pool. You create the Gateway in the AWS console.

Print the two values you need for the Gateway's inbound authorization:

macOS / Linux:

```bash
echo "Discovery URL:  https://cognito-idp.${AWS_REGION}.amazonaws.com/${GATEWAY_USER_POOL_ID}/.well-known/openid-configuration"
echo "Allowed client: ${GATEWAY_CLIENT_ID}"
```

Windows PowerShell:

```powershell
"Discovery URL:  https://cognito-idp.$($env:AWS_REGION).amazonaws.com/$($env:GATEWAY_USER_POOL_ID)/.well-known/openid-configuration"
"Allowed client: $env:GATEWAY_CLIENT_ID"
```

> **Switch to the AWS console.** Open it from the GameDay event page in
> Workshop Studio (**Open AWS console**). Check that the Region in the top
> right matches your `AWS_REGION`, then go to **Amazon Bedrock AgentCore >
> Gateways > Create gateway**.

| Setting | Value |
|---|---|
| Gateway name | `lost-colors-gateway` (the commands below look for this name) |
| Inbound auth | JSON Web Tokens (JWT) with an existing identity provider. Paste the **Discovery URL** and **Allowed client** printed above. |
| Permissions | Create and use a new service role |
| Target | The console needs at least one target to create the Gateway. Either add a dummy target and delete it later, or add the `bda` target right away with the settings from Step 11. |

Use the `agent-to-gateway` values here, not the `gateway-to-runtime` ones. The
`gateway-to-runtime` pool is for the Gateway calling the tools (Step 11).

When the Gateway is created, load its details into variables:

macOS / Linux:

```bash
export GATEWAY_ID=$(aws bedrock-agentcore-control list-gateways --query "items[?name=='lost-colors-gateway'].gatewayId | [0]" --output text)
export GATEWAY_URL=$(aws bedrock-agentcore-control get-gateway --gateway-identifier "$GATEWAY_ID" --query gatewayUrl --output text)
export GATEWAY_ROLE_ARN=$(aws bedrock-agentcore-control get-gateway --gateway-identifier "$GATEWAY_ID" --query roleArn --output text)
aws bedrock-agentcore-control get-gateway --gateway-identifier "$GATEWAY_ID" --query status --output text
env | grep -E '^GATEWAY_(ID|URL|ROLE_ARN)='
```

Windows PowerShell:

```powershell
$env:GATEWAY_ID = aws bedrock-agentcore-control list-gateways --query "items[?name=='lost-colors-gateway'].gatewayId | [0]" --output text
$env:GATEWAY_URL = aws bedrock-agentcore-control get-gateway --gateway-identifier $env:GATEWAY_ID --query gatewayUrl --output text
$env:GATEWAY_ROLE_ARN = aws bedrock-agentcore-control get-gateway --gateway-identifier $env:GATEWAY_ID --query roleArn --output text
aws bedrock-agentcore-control get-gateway --gateway-identifier $env:GATEWAY_ID --query status --output text
Get-ChildItem env:GATEWAY_ID, env:GATEWAY_URL, env:GATEWAY_ROLE_ARN
```

If you gave the Gateway a different name, replace `lost-colors-gateway` in the
first line. `GATEWAY_URL` ends in `/mcp`; you put it in the agent code in Step 10.

> **Switch to the GameDay console (Task 1, Subtask C).** When the status line
> prints `READY`, submit the value of `GATEWAY_ID` (`echo "$GATEWAY_ID"`;
> PowerShell: `$env:GATEWAY_ID`). It looks like `lost-colors-gateway-abcdefghij`.

## Step 10: Complete the agent code

The agent code in `agent/multimodal_data_agent.py` has three `<TODO>`
placeholders. Tasks 1A, 1B and 1C all ask you to fill them in and redeploy.

| Find in the code | Replace with |
|---|---|
| `provider_name="<TODO><agent-to-gateway-token>"` | `provider_name="agent-to-gateway-token"` (the credential provider from Step 6) |
| `"<TODO><MCP server/AgentCore Gateway URL>"` | Your Gateway URL in quotes, from `echo "$GATEWAY_URL"` (PowerShell: `$env:GATEWAY_URL`). For example `"https://lost-colors-gateway-abcdefghij.gateway.bedrock-agentcore.us-east-1.amazonaws.com/mcp"` |
| `system_prompt="<TODO><Instructions for the Agent>"` | The system prompt below |

### Suggested system prompt for the agent

The system prompt tells the agent who it is, which tools to use and how to
answer. Replace the `system_prompt` line so the `Agent(...)` call looks like
this. Keep the three double quotes at the start and end: the prompt itself
contains double quotes, and without them Python reports a syntax error.

```python
        agent = Agent(
            model=model,
            tools=tools,
            system_prompt="""You are an investigator tracking a string of art heists ("the lost colors"). Use the BDA tools to analyze documents, images, audio and video stored in S3 (pass s3:// URIs), the location tools to resolve places, coordinates and routes, and the flights tool to find departures by airport (IATA code) and time (ISO 8601 UTC, e.g. 2026-10-21T00:00:00Z). Cite which tool results support each conclusion and say when evidence is missing.""",
        )
```

The prompt describes the tools by what they do, so it works whatever you name
your Gateway targets in Step 11. You can adapt it, but keep the last sentence:
it makes the agent show its evidence instead of guessing.

### Redeploy and test

First check that your edited code is valid Python. This command prints nothing
if the code is fine, and the line number of the mistake if it is not:

```bash
python3 -m py_compile agent/multimodal_data_agent.py     # Windows PowerShell: python -m py_compile agent\multimodal_data_agent.py
```

Then deploy and test:

```bash
agentcore deploy --yes
agentcore invoke --runtime multimodal_data_agent --payload '{"prompt": "Which tools do you have?"}'
```

The agent now answers instead of returning the `ValidationException`. It lists
the tools of every Gateway target you have added so far; after Step 11 that is
11 tools starting with `bda___`, `location___` and `flights___`.

## Step 11: Connect the tools to the Gateway

Task 2 connects three tools to your Gateway as **targets**. Collect the values
first:

macOS / Linux:

```bash
python3 tools/runtime_URL_generator.py "$BDA_RUNTIME_ARN"
python3 tools/runtime_URL_generator.py "$LOCATION_RUNTIME_ARN"
export FLIGHT_LAMBDA_ARN=$(aws lambda list-functions --query "Functions[?starts_with(FunctionName, 'FlightQueryFunction')].FunctionArn | [0]" --output text)
echo "Flight Lambda: $FLIGHT_LAMBDA_ARN"
echo "Gateway role:  ${GATEWAY_ROLE_ARN##*/}"
```

Windows PowerShell:

```powershell
python tools/runtime_URL_generator.py $env:BDA_RUNTIME_ARN
python tools/runtime_URL_generator.py $env:LOCATION_RUNTIME_ARN
$env:FLIGHT_LAMBDA_ARN = aws lambda list-functions --query "Functions[?starts_with(FunctionName, 'FlightQueryFunction')].FunctionArn | [0]" --output text
"Flight Lambda: $env:FLIGHT_LAMBDA_ARN"
"Gateway role:  $(($env:GATEWAY_ROLE_ARN -split '/')[-1])"
```

The first two lines print the runtime URLs of the two MCP servers. Each URL
contains your account ID (after `us-east-1%3A`); check that it is your GameDay
account. A URL with the wrong account makes target creation fail with
`HTTP 404 to the initialize handshake`.

> **Switch to the AWS console:** **Amazon Bedrock AgentCore > Gateways >
> `lost-colors-gateway` > Targets > Add target**. Create one target for each
> tool below.

### Task 2A: Bedrock Data Automation (BDA) target

| Setting | Value |
|---|---|
| Target name | `bda` |
| Target type | MCP server |
| MCP endpoint | The BDA runtime URL printed above |
| Outbound auth | OAuth client, credential provider `gateway-to-runtime-token` |
| Grant type | Client credentials, no scopes |

### Task 2B: Amazon Location Service target

Same settings as BDA, with a different name and endpoint:

| Setting | Value |
|---|---|
| Target name | `location` |
| Target type | MCP server |
| MCP endpoint | The Location runtime URL printed above |
| Outbound auth | OAuth client, credential provider `gateway-to-runtime-token` |
| Grant type | Client credentials, no scopes |

### Task 2C: Flights Lambda target

Open the `FlightQueryFunction` in the Lambda console and read its code first.
It answers only one tool name, `get_flights`, and it needs two inputs:
`departureAirport` (a 3-letter IATA code such as `SYD`) and
`scheduledDeparture` (an ISO 8601 UTC time such as `2026-11-15T00:00:00Z`).

**1. Allow the Gateway to call the Lambda function.** A Lambda target uses the
Gateway's IAM service role instead of OAuth, and that role cannot invoke Lambda
functions yet.

> **Switch to the AWS console.** In the same AWS console you opened from
> Workshop Studio, type `IAM` in the search bar at the top and open **IAM**.
> Then go to **Roles** and search for `AmazonBedrockAgentCoreGatewayDefaultServiceRole`.
> The full name ends with a number, for example
> `AmazonBedrockAgentCoreGatewayDefaultServiceRole1794852061752`, and must match
> the name printed as `Gateway role` above. Open that role.

In the role, choose **Add permissions > Create inline policy > JSON**, and
paste this policy. Replace the `Resource` with your `Flight Lambda` ARN.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:FlightQueryFunction-123456789012"
    }
  ]
}
```

**2. Add the target:**

| Setting | Value |
|---|---|
| Target name | `flights` |
| Target type | Lambda ARN |
| Lambda ARN | Your `Flight Lambda` ARN |
| Outbound auth | IAM role (the Gateway service role) |
| Tool schema | Define an inline schema and paste the JSON below |

```json
[
  {
    "name": "get_flights",
    "description": "Find flights departing from an airport at or after a given time. Returns flight number, departure and arrival airports, scheduled departure and arrival times, and status.",
    "inputSchema": {
      "type": "object",
      "properties": {
        "departureAirport": {
          "type": "string",
          "description": "3-letter IATA code of the departure airport, for example SYD"
        },
        "scheduledDeparture": {
          "type": "string",
          "description": "Earliest departure time in ISO 8601 UTC format, for example 2026-11-15T00:00:00Z"
        }
      },
      "required": ["departureAirport", "scheduledDeparture"]
    }
  }
]
```

### Get the target IDs

```bash
aws bedrock-agentcore-control list-gateway-targets --gateway-identifier "$GATEWAY_ID" --query "items[].[name,targetId,status]" --output table
# Windows PowerShell: aws bedrock-agentcore-control list-gateway-targets --gateway-identifier $env:GATEWAY_ID --query "items[].[name,targetId,status]" --output table
```

Each target ID is 10 characters, for example `2DS5W5FD8S`. Wait until a target
shows `READY` before you submit it.

> **Switch to the GameDay console (Task 2).** Submit the target ID of the
> matching target for each subtask: `bda` for Subtask A, `location` for
> Subtask B and `flights` for Subtask C. Use the target ID, not the target name
> or the Gateway ID. If you added a dummy target in Step 9, you can delete it
> now.

Check that the agent sees all the tools:

```bash
agentcore invoke --runtime multimodal_data_agent --payload '{"prompt": "Which tools do you have?"}'
```

## Step 12: Run the chat frontend and investigate

Task 3 asks seven questions. You answer them by chatting with your agent in the
Streamlit frontend (you can also use `agentcore invoke`).

The frontend keeps running until you stop it, so open a **new terminal** for it
and do the new-terminal steps from Step 5. Then run:

```bash
cd UX
uv python install 3.11
uv sync --python 3.11
uv run --python 3.11 streamlit run app.py
```

Open <http://localhost:8501> in your browser. If you run it in the GameDay
Code Server instead of your own computer, open the forwarded port 8501 that
Code Server offers. In the sidebar, select your Region and the
`LostColors_multimodal_data_agent` runtime, and start chatting. Press `Ctrl+C`
in the terminal to stop the app.

> **Switch to the GameDay console (Task 3).** Read each question there, ask
> your agent in the frontend, and submit the answer in the console. Each correct
> answer unlocks the next question.

### Investigation hints

Copy the question from the GameDay console, including the answer options, and
paste it into the chat with your bucket name in front:

```text
My intelligence bucket is <your bucket name>. Use the files in it to answer this question:

<paste the question and its options here>
```

Print your bucket name with `echo "$BDA_BUCKET_NAME"` (PowerShell:
`$env:BDA_BUCKET_NAME`). Each question names the file that holds its clue, so
the agent can find it once it knows the bucket. Analyzing a file takes about 15
to 60 seconds.

## Troubleshooting

| Problem | Fix |
|---|---|
| `ExpiredToken`, `InvalidClientTokenId` or `Unable to locate credentials` | Get fresh credentials from Workshop Studio (Step 2) and paste them into this terminal. |
| `aws sts get-caller-identity` shows the wrong account | Paste the Workshop Studio credentials into this terminal again. |
| `agentcore: command not found` | Install the CLI (Step 1), then close and reopen the terminal. |
| `agentcore --version` says `No such option: --version` | Remove the old starter toolkit (Step 1). |
| `ModuleNotFoundError`, or `uv: command not found` | Activate the virtual environment (Step 3). |
| A command complains that an ID or ARN is empty or missing | Your variables are not set in this terminal. Paste the variables block from Step 5, and the runtime variables block from Step 8 if you already deployed. |
| A variable shows `None` or is empty | Your credentials expired or a step was skipped. Fix that, then paste the variables block again. |
| `no such file or directory` when a command contains `<...>` | Text in `<...>` is a placeholder. Replace it, including the `<` and `>`, with your own value. |
| `agentcore invoke` says `An error occurred when starting the runtime` | The runtime crashed at startup. Read its logs with `agentcore runtime logs --id "$AGENT_RUNTIME_ID" --since 30m` (PowerShell: `--id $env:AGENT_RUNTIME_ID`) and look at the last error. The next two rows cover the common ones. |
| The runtime logs show `SyntaxError` in `multimodal_data_agent.py` | Your code edit broke the Python syntax, often a prompt with double quotes inside `"..."`. Use triple quotes as in Step 10, check with `python3 -m py_compile agent/multimodal_data_agent.py`, then redeploy. |
| The runtime logs show `cannot import name 'streamablehttp_client'` or `No module named 'mcp.server.fastmcp'` | The `mcp` version is not pinned. Restore the `mcp` lines ending in `<2` in the three runtime `requirements.txt` files (Step 3), then `agentcore deploy --yes`. |
| `agentcore deploy` fails validation | Check that no `YOUR_` placeholders are left in `agentcore/agentcore.json` (Step 7). |
| The tools reject requests with an authorization error | Check that `agentcore/agentcore.json` uses the `gateway-to-runtime` User Pool ID and Client ID, then run `agentcore deploy` again. |
| Creating an MCP server target fails with `returned HTTP 404 to the initialize handshake` | The runtime URL contains the wrong account ID. Paste your Workshop Studio credentials, paste the runtime variables block (Step 8) again and regenerate the URL (Step 11). |
| The BDA tool reports `Asset at path S3://... does not exist` | Ask the agent again and write the path with a lowercase `s3://` prefix. |
| The flights tool fails with an access denied or not authorized error | Add the `lambda:InvokeFunction` inline policy to the Gateway's service role (Step 11). |
| The flights tool returns `Unknown tool` | The tool in the target's inline schema must be named exactly `get_flights` (Step 11). |
| The agent returns `401` or `403` errors from the Gateway | The Gateway's allowed client must be the `agent-to-gateway` client (Step 9), and `provider_name` in the agent code must be `agent-to-gateway-token` (Step 10). |
| The agent says it has no tools, or is missing some | Check that every target is `READY` (Step 11) and that the Gateway URL in the agent code ends in `/mcp`, then redeploy. Start a new chat session so the agent loads the new tool list. |
| The frontend shows no agents | Check that the sidebar Region matches the Region you deployed to, and that the frontend terminal has valid credentials. |
