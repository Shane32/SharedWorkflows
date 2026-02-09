# Configure MSDeploy with IIS User Account

This guide explains how to set up Microsoft Web Deploy (MSDeploy) on IIS using an IIS Manager user account instead of a Windows account. This approach provides better security isolation by creating deployment users that are specific to IIS and don't require Windows-level credentials.

## Overview

When using IIS Manager users for MSDeploy, you create user accounts that exist only within IIS Manager and have limited, delegated permissions. This is more secure than using Windows accounts and provides better control over deployment permissions.

## Prerequisites

- IIS with Management Service installed and running
- Administrative access to IIS Manager
- IIS Management Service Delegation feature installed

## Step 1: Add IIS Manager User

1. Open **IIS Manager**
2. Select the server node in the left panel
3. Double-click **IIS Manager Users** (under the Management section)
4. In the Actions pane, click **Add User...**
5. Enter the username and password for the deployment user
6. Click **OK** to create the user

> **Note:** IIS Manager users are specific to IIS and do not have Windows-level access. Store the password securely as it will be needed for MSDeploy authentication.

## Step 2: Configure Management Service Delegation

Management Service Delegation controls what operations remote users can perform through the IIS Management Service.

### Disable Default Rule

1. In IIS Manager, select the server node
2. Double-click **Management Service Delegation** (under the Management section)
3. Locate the default **Deploy Application** rule
4. Select it and click **Disable Rule** in the Actions pane (or right-click and select **Disable**)

> **Note:** Disabling the default rule prevents overly broad permissions. You'll create a custom rule with more specific permissions in the next step.

### Add New Custom Rule

1. Still in Management Service Delegation, click **Add Rule...** in the Actions pane
2. Select the **Deploy Application** template
3. Configure the rule settings:
   - **Provider:** `contentPath, iisApp`
   - **Actions:** `*`
   - **Path Type:** `Path Prefix`
   - **Path:** Site name and application name if applicable, such as `Default Web Site/MySite`
   - **Identity Type:** `CurrentUser`
4. Click **OK** to create the rule
5. Enter the IIS Manager username you created in Step 1 and press **OK**.
6. Ensure the new rule is **Enabled**

> **Tip:** You may need to create multiple rules if your deployment requires different types of operations (application deployment, content deployment, etc.).

## Step 3: Configure IIS Manager Permissions for the Site

1. In IIS Manager, expand the server node and select your **site** (not the application)
2. Double-click **IIS Manager Permissions** (under the Management section)
3. In the Actions pane, click **Allow User...**
4. Select **IIS Manager** (not Windows)
5. Enter the IIS Manager username you created in Step 1
6. Click **OK** to grant the user access to the site

## Step 4: Configure IIS Manager Permissions for the Application

If deploying to a specific application within the site:

1. In IIS Manager, expand the site and select your **application**
2. Double-click **IIS Manager Permissions**
3. In the Actions pane, click **Allow User...**
4. Select **IIS Manager**
5. Enter the IIS Manager username
6. Click **OK** to grant the user access to the application

> **Note:** This step is necessary if you're deploying to a specific application path (e.g., `SiteName/ApplicationName`). Skip this step if deploying directly to the site root.

## Step 5: Configure File System Permissions

MSDeploy needs file system access to write files to the deployment location. The IIS Management Service runs under the LOCAL SERVICE account, so this account needs modify permissions.

1. Open **File Explorer** and navigate to your site's physical folder (e.g., `C:\inetpub\wwwroot\YourSite`)
2. Right-click the folder and select **Properties**
3. Go to the **Security** tab
4. Click **Edit...** to modify permissions
5. Click **Add...** to add a user
6. Enter `LOCAL SERVICE` and click **Check Names**
7. Click **OK**
8. Select **LOCAL SERVICE** in the list and check **Modify** in the Allow column
9. Click **OK** to apply the permissions
10. Click **OK** to close the Properties dialog

> **Important:** Ensure that LOCAL SERVICE has Modify permissions on the entire folder structure where your application will be deployed, including any subfolders for SPA content or persisted documents.

## Step 6: Verify Configuration

After completing the setup, verify that everything is configured correctly:

1. **Test IIS Manager Connection:**
   - On a remote machine, open IIS Manager
   - Click **File** > **Connect to a Site...**
   - Enter the server name and IIS Manager credentials
   - Verify you can connect successfully

2. **Test MSDeploy Connection:**
   - Use the MSDeploy command-line tool to test the connection:
     ```cmd
     msdeploy -verb:dump -source:contentPath="YourSiteName",computerName="https://server:8172/msdeploy.axd?site=YourSiteName",userName="IISManagerUser",password="password",authType=Basic -allowUntrusted
     ```
   - You should see a successful connection and site content listing

3. **Test Deployment:**
   - Perform a test deployment using your CI/CD workflow
   - Verify files are deployed correctly
   - Check for any permission-related errors

## Understanding MSDeploy URL Construction

The MSDeploy server URL follows a specific format that is important to understand:

```text
https://server-name:8172/msdeploy.axd?site=SiteName
```

**Important:** The `site` parameter in the URL query string should contain **only the site name**, not the application path. This is a critical distinction:

- ✅ **Correct:** `https://server:8172/msdeploy.axd?site=Default Web Site`
- ❌ **Incorrect:** `https://server:8172/msdeploy.axd?site=Default Web Site/MyApp`

The application path is specified separately in the deployment target (e.g., in the `msdeploy_site_name` parameter as `SiteName/ApplicationName`), not in the MSDeploy URL itself.

### Why This Matters

The `/msdeploy.axd` handler is registered at the site level in IIS. When you specify the site in the query string, MSDeploy:

1. Connects to the IIS Management Service on the specified port (typically 8172)
2. Uses the site name from the query string to identify which site's context to use
3. Authenticates using the provided IIS Manager credentials
4. Then applies the specific deployment target (site or site/application) as specified in the deployment command

If you incorrectly include the application path in the URL query string, the connection will fail because IIS won't find a site with that combined name.

### Example Workflow Configuration

When configuring the [Deploy with MSDeploy workflow](deploy-msdeploy.md), you must specify the site name separately from the application path:

**For deploying to a site root:**
```yaml
with:
  msdeploy_server_url: https://server:8172/msdeploy.axd?site=Default Web Site
  msdeploy_site_name: Default Web Site
```

**For deploying to an application within a site:**
```yaml
with:
  msdeploy_server_url: https://server:8172/msdeploy.axd?site=Default Web Site
  msdeploy_site_name: Default Web Site/MyApplication
```

Note how in both cases:
- The `msdeploy_server_url` parameter contains only the site name in `?site=Default Web Site`
- The `msdeploy_site_name` parameter specifies the deployment target, which may include the application path (`Default Web Site/MyApplication`)
