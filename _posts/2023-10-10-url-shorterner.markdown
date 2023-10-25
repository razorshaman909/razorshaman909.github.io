---
layout: post
title:  "Short Link Generator"
date:   2023-10-10 19:43:59 +0800
categories: personal project
---
To use this Short Link Generator, enter a long link in the form below and click the "Generate Short Link" button. The tool will provide you with a shorter link. The tool works based on the created issues on the respective repository, returning the issue title as the link. Do mind that accessing the short link have a 60 daily rate limit.

<form id="linkForm">
    <label for="longLink">Enter a Long Link:</label>
    <input type="text" id="longLink" name="longLink" required>
    <button type="submit">Generate Short Link</button>
</form>

Short Link: <span id="shortLink"></span>

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