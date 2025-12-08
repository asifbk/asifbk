<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Stats Generator - Georgia Font</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Georgia, serif;
        }
        
        body {
            background: #f5f5f5;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        h1 {
            font-family: Georgia, serif;
            text-align: center;
            color: #333;
            margin-bottom: 30px;
            font-size: 2.5em;
        }
        
        .input-section {
            text-align: center;
            margin-bottom: 30px;
        }
        
        input {
            font-family: Georgia, serif;
            padding: 12px 20px;
            font-size: 16px;
            border: 2px solid #ddd;
            border-radius: 5px;
            width: 300px;
            margin-right: 10px;
        }
        
        button {
            font-family: Georgia, serif;
            padding: 12px 30px;
            font-size: 16px;
            background: #0366d6;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.3s;
        }
        
        button:hover {
            background: #0256b8;
        }
        
        #stats-canvas {
            display: none;
        }
        
        .canvas-container {
            margin: 20px 0;
            text-align: center;
        }
        
        .preview {
            border: 1px solid #ddd;
            border-radius: 10px;
            max-width: 100%;
            margin: 20px auto;
            display: block;
        }
        
        .download-btn {
            background: #28a745;
            margin: 10px 5px;
        }
        
        .download-btn:hover {
            background: #218838;
        }
        
        .loading {
            text-align: center;
            font-family: Georgia, serif;
            color: #666;
            font-size: 18px;
            padding: 20px;
        }
        
        .error {
            color: #d73a49;
            text-align: center;
            font-family: Georgia, serif;
            padding: 10px;
        }
        
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
        
        .instructions {
            background: #f6f8fa;
            padding: 20px;
            border-radius: 8px;
            margin: 30px 0;
            font-family: Georgia, serif;
        }
        
        .instructions h3 {
            font-family: Georgia, serif;
            color: #0366d6;
            margin-bottom: 10px;
        }
        
        .instructions ol {
            margin-left: 20px;
        }
        
        .instructions li {
            margin: 8px 0;
            line-height: 1.6;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📊 GitHub Stats Generator</h1>
        <p style="text-align: center; font-family: Georgia, serif; color: #666; margin-bottom: 20px;">
            Generate beautiful GitHub statistics with Georgia font
        </p>
        
        <div class="input-section">
            <input type="text" id="username" placeholder="Enter GitHub username" value="asifbk">
            <button onclick="generateStats()">Generate Stats</button>
        </div>
        
        <div id="loading" class="loading" style="display: none;">
            Loading your GitHub statistics...
        </div>
        
        <div id="error" class="error"></div>
        
        <div id="preview-section" style="display: none;">
            <h2 style="font-family: Georgia, serif; text-align: center; margin: 20px 0;">Preview</h2>
            <canvas id="stats-canvas"></canvas>
            <img id="preview" class="preview" alt="Stats Preview">
            <div style="text-align: center;">
                <button class="download-btn" onclick="downloadImage()">📥 Download Image</button>
            </div>
        </div>
        
        <div class="instructions">
            <h3>📋 How to Use:</h3>
            <ol>
                <li>Enter your GitHub username and click "Generate Stats"</li>
                <li>Wait for the statistics to load</li>
                <li>Download the generated image</li>
                <li>Upload the image to your GitHub repository (create an <code>assets</code> folder)</li>
                <li>Add to your README.md: <code>![GitHub Stats](./assets/github-stats.png)</code></li>
            </ol>
            
            <h3 style="margin-top: 20px;">⚠️ Important Note:</h3>
            <p style="margin-top: 10px;">
                GitHub README files do not support custom fonts. The external stat card services also cannot be customized to use Georgia font. 
                This tool creates a custom image with Georgia font that you can upload to your repository and display as an image.
            </p>
        </div>
    </div>

    <script>
        async function generateStats() {
            const username = document.getElementById('username').value.trim();
            if (!username) {
                document.getElementById('error').textContent = 'Please enter a username';
                return;
            }
            
            document.getElementById('loading').style.display = 'block';
            document.getElementById('error').textContent = '';
            document.getElementById('preview-section').style.display = 'none';
            
            try {
                const userResponse = await fetch(`https://api.github.com/users/${username}`);
                if (!userResponse.ok) throw new Error('User not found');
                const userData = await userResponse.json();
                
                const reposResponse = await fetch(`https://api.github.com/users/${username}/repos?per_page=100`);
                const reposData = await reposResponse.json();
                
                const totalStars = reposData.reduce((sum, repo) => sum + repo.stargazers_count, 0);
                const totalForks = reposData.reduce((sum, repo) => sum + repo.forks_count, 0);
                
                const languageCounts = {};
                reposData.forEach(repo => {
                    if (repo.language) {
                        languageCounts[repo.language] = (languageCounts[repo.language] || 0) + 1;
                    }
                });
                
                const topLanguages = Object.entries(languageCounts)
                    .sort((a, b) => b[1] - a[1])
                    .slice(0, 5);
                
                drawStats({
                    name: userData.name || userData.login,
                    username: userData.login,
                    followers: userData.followers,
                    following: userData.following,
                    repos: userData.public_repos,
                    stars: totalStars,
                    forks: totalForks,
                    languages: topLanguages
                });
                
                document.getElementById('loading').style.display = 'none';
                document.getElementById('preview-section').style.display = 'block';
                
            } catch (error) {
                document.getElementById('loading').style.display = 'none';
                document.getElementById('error').textContent = 'Error: ' + error.message;
            }
        }
        
        function drawStats(data) {
            const canvas = document.getElementById('stats-canvas');
            const ctx = canvas.getContext('2d');
            
            canvas.width = 900;
            canvas.height = 600;
            
            // Background
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Set font
            ctx.font = 'bold 36px Georgia';
            ctx.fillStyle = '#333';
            ctx.textAlign = 'center';
            ctx.fillText('📊 GitHub Statistics', canvas.width / 2, 60);
            
            // Name
            ctx.font = 'bold 28px Georgia';
            ctx.fillText(data.name, canvas.width / 2, 110);
            
            ctx.font = '20px Georgia';
            ctx.fillStyle = '#666';
            ctx.fillText('@' + data.username, canvas.width / 2, 140);
            
            // Stats boxes
            const stats = [
                { label: 'Followers', value: data.followers, color: '#0366d6' },
                { label: 'Repositories', value: data.repos, color: '#28a745' },
                { label: 'Total Stars', value: data.stars, color: '#ffd33d' },
                { label: 'Following', value: data.following, color: '#6f42c1' }
            ];
            
            const boxWidth = 180;
            const boxHeight = 100;
            const spacing = 20;
            const startX = (canvas.width - (stats.length * boxWidth + (stats.length - 1) * spacing)) / 2;
            const startY = 180;
            
            stats.forEach((stat, index) => {
                const x = startX + index * (boxWidth + spacing);
                const y = startY;
                
                // Box background
                ctx.fillStyle = stat.color + '20';
                ctx.fillRect(x, y, boxWidth, boxHeight);
                
                // Border
                ctx.strokeStyle = stat.color;
                ctx.lineWidth = 3;
                ctx.strokeRect(x, y, boxWidth, boxHeight);
                
                // Value
                ctx.fillStyle = stat.color;
                ctx.font = 'bold 32px Georgia';
                ctx.textAlign = 'center';
                ctx.fillText(stat.value, x + boxWidth / 2, y + 50);
                
                // Label
                ctx.fillStyle = '#666';
                ctx.font = '16px Georgia';
                ctx.fillText(stat.label, x + boxWidth / 2, y + 80);
            });
            
            // Languages section
            ctx.fillStyle = '#333';
            ctx.font = 'bold 24px Georgia';
            ctx.textAlign = 'center';
            ctx.fillText('🔥 Top Languages', canvas.width / 2, 340);
            
            const langColors = ['#0366d6', '#28a745', '#ffd33d', '#6f42c1', '#d73a49'];
            const langStartY = 370;
            const langHeight = 30;
            
            data.languages.forEach(([lang, count], index) => {
                const y = langStartY + index * (langHeight + 10);
                
                // Language name
                ctx.fillStyle = '#333';
                ctx.font = '18px Georgia';
                ctx.textAlign = 'left';
                ctx.fillText(lang, 150, y + 20);
                
                // Bar
                const barWidth = (count / data.languages[0][1]) * 400;
                ctx.fillStyle = langColors[index % langColors.length];
                ctx.fillRect(300, y, barWidth, langHeight);
                
                // Count
                ctx.fillStyle = '#666';
                ctx.font = '16px Georgia';
                ctx.textAlign = 'left';
                ctx.fillText(count + ' repos', 710, y + 20);
            });
            
            // Convert to preview
            const preview = document.getElementById('preview');
            preview.src = canvas.toDataURL();
        }
        
        function downloadImage() {
            const canvas = document.getElementById('stats-canvas');
            const link = document.createElement('a');
            link.download = 'github-stats.png';
            link.href = canvas.toDataURL();
            link.click();
        }
        
        // Auto-generate on load
        window.onload = () => generateStats();
    </script>
</body>
</html>
