<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>YouTube Similar Video Finder</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      padding: 2rem;
      text-align: center;
    }

    input, button {
      padding: 10px;
      font-size: 16px;
      width: 80%;
      max-width: 500px;
      margin: 10px auto;
    }

    .video-list {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 1rem;
      margin-top: 2rem;
    }

    .video-card {
      background: #fff;
      border-radius: 12px;
      width: 300px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      transition: transform 0.2s;
    }

    .video-card:hover {
      transform: scale(1.05);
    }

    .video-card img {
      width: 100%;
      border-radius: 12px 12px 0 0;
    }

    .video-title {
      padding: 10px;
    }
  </style>
</head>
<body>
  <h1>🔎 YouTube Similar Video Finder</h1>
  <input id="videoUrl" type="text" placeholder="Paste YouTube Video URL" />
  <br />
  <button onclick="findSimilarVideos()">Find Similar</button>

  <div class="video-list" id="results"></div>

  <script>
    const apiKey = "AIzaSyDg4MrLZVDZM78bCnVFD0V2fi2KU8_lsjA";

    function extractVideoID(url) {
      const match = url.match(/(?:v=|\/)([0-9A-Za-z_-]{11})/);
      return match ? match[1] : null;
    }

    async function findSimilarVideos() {
      const videoUrl = document.getElementById("videoUrl").value;
      const videoId = extractVideoID(videoUrl);
      const container = document.getElementById("results");
      container.innerHTML = "Loading...";

      if (!videoId) {
        container.innerHTML = "<p>Invalid YouTube URL.</p>";
        return;
      }

      try {
        // Search similar videos
        const searchRes = await fetch(
          `https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&maxResults=15&relatedToVideoId=${videoId}&key=${apiKey}`
        );
        const searchData = await searchRes.json();
        const videoIds = searchData.items.map(item => item.id.videoId).join(",");

        // Get video durations
        const videoRes = await fetch(
          `https://www.googleapis.com/youtube/v3/videos?part=snippet,contentDetails&id=${videoIds}&key=${apiKey}`
        );
        const videoData = await videoRes.json();

        // Filter results
        const validVideos = videoData.items.filter(item => {
          const title = item.snippet.title.toLowerCase();
          const duration = parseISO8601Duration(item.contentDetails.duration);
          return !title.includes("#short") && duration >= 60;
        });

        // Display
        container.innerHTML = "";
        if (validVideos.length === 0) {
          container.innerHTML = "<p>No suitable videos found.</p>";
        }

        validVideos.forEach(video => {
          const thumb = video.snippet.thumbnails.high.url;
          const title = video.snippet.title;
          const link = `https://www.youtube.com/watch?v=${video.id}`;

          const card = document.createElement("div");
          card.className = "video-card";
          card.innerHTML = `
            <a href="${link}" target="_blank">
              <img src="${thumb}" alt="${title}" />
            </a>
            <div class="video-title">${title}</div>
          `;
          container.appendChild(card);
        });
      } catch (error) {
        container.innerHTML = "<p>Error fetching videos.</p>";
        console.error(error);
      }
    }

    // Convert YouTube ISO 8601 duration (e.g. PT5M30S) to seconds
    function parseISO8601Duration(iso) {
      const match = iso.match(/PT(?:(\d+)M)?(?:(\d+)S)?/);
      const minutes = parseInt(match[1] || "0");
      const seconds = parseInt(match[2] || "0");
      return minutes * 60 + seconds;
    }
  </script>
</body>
</html>
