---
layout: post
title:  "Short Link Generator"
date:   2023-10-10 19:43:59 +0800
categories: personal project
---
To use this Short Link Generator, enter a long link in the form below and click the "Generate Short Link" button. The tool will provide you with a shorter link. The tool works based on the created issues on the respective repository, returning the issue title as the link. Do mind that accessing the short link have a 60 daily rate limit per ip address.

<form id="linkForm">
    <label for="longLink">Enter a Long Link:</label>
    <input type="text" id="longLink" name="longLink" required>
    <button type="submit">Generate Short Link</button>
</form>

Short Link: <span id="shortLink"></span>


Alternative Google Sheet short link generator, first autorize for Google OAuth, then proceed with input with long link to be shortened and custom short link to be appended as extension. The link can then be used using a alternatively hosted domain for redirection.

<button id="authorize_button" onclick="handleAuthClick()">Authorize</button>
<button id="signout_button" onclick="handleSignoutClick()">Sign Out</button>

<form>
  <label for="longl">Longlink:</label><br>
  <input type="text" id="longl" name="longl"><br>
  <label for="shortl">Shortlink:</label><br>
  <input type="text" id="shortl" name="shortl"><br>
</form>
<button id="submission" onclick="submitdata(shortl.value,longl.value)">Submit</button>


<script>
    document.getElementById('linkForm').addEventListener('submit', function (e) {
    e.preventDefault();

    const longLink = document.getElementById('longLink').value;

    // Replace 'YOUR_ACCESS_TOKEN' and 'OWNER/REPO' with your GitHub access token and the repository you want to create an issue in.
    const accessToken = 'YOUR_ACCESS_TOKEN';
    const repo = 'razorshaman909/razorshaman909.github.io';

    createGitHubIssue(accessToken, repo, longLink)
        .then((response) => {
            if (response.status === 201) {
                return response.json(); 
                /**document.getElementById('shortLink').textContent = 'Issue created successfully!'; **/
            } else {
                document.getElementById('shortLink').textContent = 'Short link/Issue creation failed. Check your access token and repository.';
            }
        })
        .then((data) => {
            if (data && data.number) {
                const issueNumber = data.number
                /**console.log(issueNumber)**/
                document.getElementById('shortLink').innerHTML  = `https://razorshaman909.github.io/${issueNumber}`;
            }
        }
        )
        .catch((error) => {
            console.error('Error:', error);
        });
    });

    function createGitHubIssue(accessToken, repo, longLink) {
        const url = `https://api.github.com/repos/${repo}/issues`;
        const issueData = {
            title: `${longLink}`,
            body: ` `,
        };

        return fetch(url, {
            method: 'POST',
            headers: {
                Authorization: `token ${accessToken}`,
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(issueData),
        });
}

</script>
<script>
    // Replace 'SPREADSHEET_ID', 'CLIENT_ID', 'API_KEY' with yours own.
    const SPREADSHEET_ID = 'SPREADSHEET_ID';
    const CLIENT_ID = 'CLIENT_ID';
    const API_KEY = 'API_KEY';
    const SCOPES = 'https://www.googleapis.com/auth/spreadsheets';
    const DISCOVERY_DOC = 'https://sheets.googleapis.com/$discovery/rest?version=v4';

    let tokenClient;
    let gapiInited = false;
    let gisInited = false;

    document.getElementById('authorize_button').style.visibility = 'hidden';
    document.getElementById('signout_button').style.visibility = 'hidden';
    
    function clearbox(){
      document.getElementById('content').innerText = '';
    }

    function submitdata(a,b,){
      let values = [[a,b]];
      console.log(values);
      const body = {
        'values': values,
      };
      try {
        gapi.client.sheets.spreadsheets.values.append({
          spreadsheetId: SPREADSHEET_ID,
          range: 'Sheet1',
          valueInputOption: 'RAW',
          resource: body,
        }).then((response) => {
          const result = response.result;
          console.log(`${result.data.updates.updatedCells} cells appended.`);
          if (callback) callback(response);
        });
      } catch (err) {
        document.getElementById('content').innerText = err.message;
        return;
      }
    }

    function gapiLoaded() {
        gapi.load('client', initializeGapiClient);
      }

    async function initializeGapiClient() {
        await gapi.client.init({
          apiKey: API_KEY,
          discoveryDocs: [DISCOVERY_DOC],
        });
        gapiInited = true;
        maybeEnableButtons();
      }
    
    function gisLoaded() {
        tokenClient = google.accounts.oauth2.initTokenClient({
          client_id: CLIENT_ID,
          scope: SCOPES,
          callback: '', // defined later
        });
        gisInited = true;
        maybeEnableButtons();
      }

    function maybeEnableButtons() {
        if (gapiInited && gisInited) {
            document.getElementById('authorize_button').style.visibility = 'visible';
        }
    }

    function handleAuthClick() {
    tokenClient.callback = async (resp) => {
        if (resp.error !== undefined) {
        throw (resp);
        }
        document.getElementById('signout_button').style.visibility = 'visible';
        document.getElementById('authorize_button').innerText = 'Refresh';
        await listMajors();
    };

    if (gapi.client.getToken() === null) {
        // Prompt the user to select a Google Account and ask for consent to share their data
        // when establishing a new session.
        tokenClient.requestAccessToken({prompt: 'consent'});
    } else {
        // Skip display of account chooser and consent dialog for an existing session.
        tokenClient.requestAccessToken({prompt: ''});
    }
    }

    function handleSignoutClick() {
        const token = gapi.client.getToken();
        if (token !== null) {
          google.accounts.oauth2.revoke(token.access_token);
          gapi.client.setToken('');
          document.getElementById('content').innerText = '';
          document.getElementById('authorize_button').innerText = 'Authorize';
          document.getElementById('signout_button').style.visibility = 'hidden';
        }
      }
    
    async function listMajors() {
        let response;
        try {
          // Fetch first 10 files
          // response = await gapi.client.sheets.spreadsheets.values.get({
          //   spreadsheetId: '1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms',
          //   range: 'Class Data!A2:E',
          // });
        } catch (err) {
          document.getElementById('content').innerText = err.message;
          return;
        }
        const range = response.result;
        if (!range || !range.values || range.values.length == 0) {
          document.getElementById('content').innerText = 'No values found.';
          return;
        }
        // Flatten to string to display
        // const output = range.values.reduce(
        //     (str, row) => `${str}${row[0]}, ${row[4]}\n`,
        //     'Name, Major:\n');
        document.getElementById('content').innerText = "Authorised";
      }
</script>
<script async defer src="https://apis.google.com/js/api.js" onload="gapiLoaded()"></script>
<script async defer src="https://accounts.google.com/gsi/client" onload="gisLoaded()"></script>