---
layout: post
title: 乐队网页设计
date: 2025-11-05 08:40 +0000
categories: [Program]
tag: [Python, Web Design, backend, sync, async, Database]
---

在这个博客中，我们将要着重学习后端设计，并了解数据库的基本使用。
注：为简化学习，将前端框架置于此，不再单独讲解。（后续讲解前端调用后端API流程时会提到）

该项目提供三个页面（可跳转）

- **index.html**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BanG Dream! 乐队查询系统</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        h1 {
            color: #333;
            font-size: 2.5em;
            margin-bottom: 10px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .search-section {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 30px;
        }
        
        .search-form {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        input[type="text"] {
            flex: 1;
            padding: 12px;
            border: 2px solid #e9ecef;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        
        input[type="text"]:focus {
            outline: none;
            border-color: #667eea;
        }
        
        button {
            padding: 12px 24px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            transition: transform 0.2s;
        }
        
        button:hover {
            transform: translateY(-2px);
        }
        
        .bands-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .band-card {
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s;
            cursor: pointer;
            border-left: 4px solid #667eea;
        }
        
        .band-card:hover {
            transform: translateY(-5px);
        }
        
        .band-name {
            font-size: 1.4em;
            color: #333;
            margin-bottom: 10px;
            font-weight: bold;
        }
        
        .band-description {
            color: #666;
            line-height: 1.5;
        }
        
        .navigation {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 30px;
        }
        
        .nav-btn {
            padding: 10px 20px;
            text-decoration: none;
            background: #6c757d;
            color: white;
            border-radius: 5px;
            transition: background 0.3s;
        }
        
        .nav-btn:hover {
            background: #5a6268;
        }
        
        .nav-btn.primary {
            background: #667eea;
        }
        
        .nav-btn.primary:hover {
            background: #5a6fd8;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🎵 BanG Dream! 乐队查询系统</h1>
            <p>探索各个乐队的音乐世界</p>
        </header>
        
        <section class="search-section">
            <h2>乐队查询</h2>
            <div class="search-form">
                <input type="text" id="bandName" placeholder="输入乐队名称，如：MyGO!!!!!">
                <button onclick="searchBand()">搜索乐队</button>
            </div>
            
            <div class="bands-grid" id="bandsContainer">
                <!-- 乐队卡片将通过JavaScript动态生成 -->
            </div>
        </section>
        
        <div class="navigation">
            <a href="music.html" class="nav-btn primary">歌曲管理</a>
        </div>
    </div>

    <script>
        // 页面加载时获取所有乐队信息
        document.addEventListener('DOMContentLoaded', function() {
            fetchBands();
        });

        // 获取所有乐队信息
        async function fetchBands() {
            try {
                console.log('正在获取乐队信息...');
                const response = await fetch('http://localhost:8000/api/bands');
                console.log('响应状态:', response.status);
                
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                
                const bands = await response.json();
                console.log('获取到的乐队数据:', bands);
                displayBands(bands);
            } catch (error) {
                console.error('获取乐队信息失败:', error);
                document.getElementById('bandsContainer').innerHTML = 
                    '<p style="color: red; text-align: center;">获取乐队信息失败，请检查后端服务是否启动</p>';
            }
        }

        // 搜索特定乐队
        async function searchBand() {
            const bandName = document.getElementById('bandName').value.trim();
            if (!bandName) {
                alert('请输入乐队名称');
                return;
            }

            try {
                console.log('正在搜索乐队:', bandName);
                const response = await fetch(`http://localhost:8000/api/bands?name=${encodeURIComponent(bandName)}`);
                console.log('搜索响应状态:', response.status);
                
                if (response.status === 404) {
                    alert('未找到该乐队信息');
                    return;
                }
                
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                
                const band = await response.json();
                console.log('搜索到的乐队数据:', band);
                displayBands(Array.isArray(band) ? band : [band]);
            } catch (error) {
                console.error('搜索乐队失败:', error);
                alert('搜索失败，请稍后重试');
            }
        }

        // 显示乐队列表
        function displayBands(bands) {
            const container = document.getElementById('bandsContainer');
            console.log('显示乐队数据:', bands);
            
            if (!bands || bands.length === 0) {
                container.innerHTML = '<p style="text-align: center; color: #666;">未找到乐队信息</p>';
                return;
            }

            // 转义乐队名称中的特殊字符
            container.innerHTML = bands.map(band => `
                <div class="band-card" onclick="viewBandDetail('${band.name.replace(/'/g, "\\'")}')">
                    <div class="band-name">${band.name}</div>
                    <div class="band-description">${band.description || '暂无描述'}</div>
                </div>
            `).join('');
        }

        // 查看乐队详情
        function viewBandDetail(bandName) {
            console.log('查看乐队详情:', bandName);
            window.location.href = `band.html?name=${encodeURIComponent(bandName)}`;
        }
    </script>
</body>
</html>
```

- **band.html**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>乐队详情 - BanG Dream!</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        }
        
        .back-btn {
            display: inline-block;
            padding: 10px 20px;
            background: #6c757d;
            color: white;
            text-decoration: none;
            border-radius: 5px;
            margin-bottom: 20px;
            transition: background 0.3s;
        }
        
        .back-btn:hover {
            background: #5a6268;
        }
        
        .band-header {
            text-align: center;
            margin-bottom: 30px;
            padding: 30px;
            background: linear-gradient(45deg, #f8f9fa, #e9ecef);
            border-radius: 10px;
        }
        
        .band-name {
            font-size: 3em;
            color: #333;
            margin-bottom: 15px;
            font-weight: bold;
        }
        
        .band-description {
            font-size: 1.2em;
            color: #666;
            line-height: 1.6;
            max-width: 800px;
            margin: 0 auto;
        }
        
        .songs-section {
            margin-top: 40px;
        }
        
        .section-title {
            font-size: 1.8em;
            color: #333;
            margin-bottom: 20px;
            border-bottom: 2px solid #667eea;
            padding-bottom: 10px;
        }
        
        .songs-grid {
            display: grid;
            gap: 15px;
        }
        
        .song-card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
            border-left: 4px solid #764ba2;
            transition: transform 0.2s;
        }
        
        .song-card:hover {
            transform: translateX(5px);
        }
        
        .song-title {
            font-size: 1.3em;
            color: #333;
            margin-bottom: 8px;
            font-weight: bold;
        }
        
        .song-author {
            color: #666;
            margin-bottom: 10px;
            font-style: italic;
        }
        
        .song-lyrics {
            color: #777;
            line-height: 1.5;
            max-height: 100px;
            overflow: hidden;
            position: relative;
        }
        
        .song-lyrics::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            height: 20px;
            background: linear-gradient(transparent, white);
        }
        
        .loading {
            text-align: center;
            padding: 40px;
            color: #666;
        }
        
        .error {
            text-align: center;
            padding: 40px;
            color: #dc3545;
            background: #f8d7da;
            border-radius: 8px;
            margin: 20px 0;
        }
        
        .pagination {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 30px;
        }
        
        .page-btn {
            padding: 8px 16px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.3s;
        }
        
        .page-btn:hover:not(:disabled) {
            background: #5a6fd8;
        }
        
        .page-btn:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <div class="container">
        <a href="index.html" class="back-btn">← 返回首页</a>
        
        <div id="bandInfo" class="band-header">
            <div class="loading">加载中...</div>
        </div>
        
        <section class="songs-section">
            <h2 class="section-title">🎵 乐队歌曲</h2>
            <div id="songsContainer" class="songs-grid">
                <div class="loading">加载歌曲中...</div>
            </div>
            
            <div class="pagination" id="pagination">
                <!-- 分页按钮将通过JavaScript动态生成 -->
            </div>
        </section>
    </div>

    <script>
        // 获取URL参数
        function getQueryParam(name) {
            const urlParams = new URLSearchParams(window.location.search);
            return urlParams.get(name);
        }

        // 页面加载时获取乐队详情
        document.addEventListener('DOMContentLoaded', function() {
            const bandName = getQueryParam('name');
            if (bandName) {
                fetchBandDetail(bandName);
                fetchBandSongs(bandName);
            } else {
                document.getElementById('bandInfo').innerHTML = 
                    '<div class="error">未指定乐队名称</div>';
            }
        });

        // 获取乐队详情
        async function fetchBandDetail(bandName) {
            try {
                console.log(`正在获取乐队详情: ${bandName}`);
                const response = await fetch(`http://localhost:8000/api/bands?name=${encodeURIComponent(bandName)}`);
                
                console.log('响应状态:', response.status);
                
                if (response.status === 404) {
                    document.getElementById('bandInfo').innerHTML = 
                        '<div class="error">未找到该乐队信息</div>';
                    return;
                }
                
                if (!response.ok) {
                    throw new Error(`HTTP错误! 状态: ${response.status}`);
                }
                
                const band = await response.json();
                console.log('获取到的乐队数据:', band);
                displayBandDetail(band);
            } catch (error) {
                console.error('获取乐队详情失败:', error);
                document.getElementById('bandInfo').innerHTML = 
                    '<div class="error">获取乐队信息失败: ' + error.message + '</div>';
            }
        }

        // 显示乐队详情
        function displayBandDetail(band) {
            const bandInfo = document.getElementById('bandInfo');
            
            // 检查band是否为数组
            if (Array.isArray(band) && band.length > 0) {
                // 如果返回的是数组，取第一个元素
                band = band[0];
            }
            
            // 检查band对象是否有效
            if (!band || !band.name) {
                bandInfo.innerHTML = '<div class="error">乐队数据格式错误</div>';
                return;
            }
            
            bandInfo.innerHTML = `
                <div class="band-name">${escapeHtml(band.name)}</div>
                <div class="band-description">${escapeHtml(band.description || '暂无描述')}</div>
            `;
            
            // 更新页面标题
            document.title = `${band.name} - BanG Dream!`;
        }

        // HTML转义函数，防止XSS攻击
        function escapeHtml(unsafe) {
            if (unsafe === null || unsafe === undefined) return '';
            return unsafe
                .toString()
                .replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;")
                .replace(/"/g, "&quot;")
                .replace(/'/g, "&#039;");
        }

        let currentPage = 1;
        const pageSize = 5;

        // 获取乐队歌曲（带分页）
        async function fetchBandSongs(bandName, page = 1) {
            try {
                console.log(`正在获取第${page}页的歌曲，乐队: ${bandName}`);
                const response = await fetch(
                    `http://localhost:8000/api/songs?band=${encodeURIComponent(bandName)}&page_index=${page}&page_size=${pageSize}`
                );
                
                if (!response.ok) {
                    throw new Error(`HTTP错误! 状态: ${response.status}`);
                }
                
                const data = await response.json();
                console.log('获取到的歌曲数据:', data);
                displaySongs(data.songs || data); // 兼容两种可能的响应格式
                setupPagination(data.total || (data.songs ? data.songs.length : 0), page, bandName);
                currentPage = page;
            } catch (error) {
                console.error('获取歌曲失败:', error);
                document.getElementById('songsContainer').innerHTML = 
                    '<div class="error">获取歌曲信息失败: ' + error.message + '</div>';
            }
        }

        // 显示歌曲列表
        function displaySongs(songs) {
            const container = document.getElementById('songsContainer');
            
            if (!songs || songs.length === 0) {
                container.innerHTML = '<div class="error">该乐队暂无歌曲</div>';
                return;
            }

            container.innerHTML = songs.map(song => `
                <div class="song-card">
                    <div class="song-title">${escapeHtml(song.title || '未知歌曲')}</div>
                    <div class="song-author">作者: ${escapeHtml(song.author || '未知')}</div>
                    <div class="song-lyrics">${escapeHtml(song.lyrics || '暂无歌词')}</div>
                </div>
            `).join('');
        }

        // 设置分页
        function setupPagination(total, currentPage, bandName) {
            const totalPages = Math.ceil(total / pageSize);
            const pagination = document.getElementById('pagination');
            
            if (totalPages <= 1) {
                pagination.innerHTML = '';
                return;
            }

            pagination.innerHTML = `
                <button class="page-btn" onclick="fetchBandSongs('${escapeHtml(bandName)}', ${currentPage - 1})" 
                    ${currentPage <= 1 ? 'disabled' : ''}>上一页</button>
                <span>第 ${currentPage} 页 / 共 ${totalPages} 页</span>
                <button class="page-btn" onclick="fetchBandSongs('${escapeHtml(bandName)}', ${currentPage + 1})" 
                    ${currentPage >= totalPages ? 'disabled' : ''}>下一页</button>
            `;
        }
    </script>
</body>
</html>
```

- **music.html**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>歌曲管理 - BanG Dream!</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        }
        
        .back-btn {
            display: inline-block;
            padding: 10px 20px;
            background: #6c757d;
            color: white;
            text-decoration: none;
            border-radius: 5px;
            margin-bottom: 20px;
            transition: background 0.3s;
        }
        
        .back-btn:hover {
            background: #5a6268;
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        h1 {
            color: #333;
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        
        .management-section {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-top: 30px;
        }
        
        .form-section, .list-section {
            background: #f8f9fa;
            padding: 25px;
            border-radius: 10px;
        }
        
        .section-title {
            font-size: 1.5em;
            color: #333;
            margin-bottom: 20px;
            border-bottom: 2px solid #667eea;
            padding-bottom: 10px;
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            color: #333;
            font-weight: 500;
        }
        
        input, textarea, select {
            width: 100%;
            padding: 12px;
            border: 2px solid #e9ecef;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        
        input:focus, textarea:focus, select:focus {
            outline: none;
            border-color: #667eea;
        }
        
        textarea {
            height: 120px;
            resize: vertical;
        }
        
        .btn-group {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }
        
        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.3s;
        }
        
        .btn-primary {
            background: #667eea;
            color: white;
        }
        
        .btn-primary:hover {
            background: #5a6fd8;
            transform: translateY(-2px);
        }
        
        .btn-secondary {
            background: #6c757d;
            color: white;
        }
        
        .btn-secondary:hover {
            background: #5a6268;
        }
        
        .btn-danger {
            background: #dc3545;
            color: white;
        }
        
        .btn-danger:hover {
            background: #c82333;
        }
        
        .songs-list {
            max-height: 500px;
            overflow-y: auto;
        }
        
        .song-item {
            background: white;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
            border-left: 4px solid #764ba2;
        }
        
        .song-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        
        .song-title {
            font-size: 1.2em;
            color: #333;
            font-weight: bold;
        }
        
        .song-band {
            background: #667eea;
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.8em;
        }
        
        .song-author {
            color: #666;
            margin-bottom: 10px;
        }
        
        .song-lyrics {
            color: #777;
            line-height: 1.4;
            max-height: 60px;
            overflow: hidden;
        }
        
        .song-actions {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }
        
        .action-btn {
            padding: 6px 12px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        
        .search-section {
            margin-bottom: 20px;
        }
        
        .search-form {
            display: flex;
            gap: 10px;
            margin-bottom: 10px;
        }
        
        .filter-section {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        .filter-group {
            flex: 1;
        }
        
        .message {
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            text-align: center;
        }
        
        .message.success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .message.error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .pagination {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
        }
        
        .page-btn {
            padding: 8px 16px;
            border: 1px solid #ddd;
            background: white;
            cursor: pointer;
            border-radius: 4px;
        }
        
        .page-btn.active {
            background: #667eea;
            color: white;
            border-color: #667eea;
        }
    </style>
</head>
<body>
    <div class="container">
        <a href="index.html" class="back-btn">← 返回首页</a>
        
        <header>
            <h1>🎵 歌曲管理系统</h1>
            <p>管理所有乐队的歌曲信息</p>
        </header>
        
        <div id="message"></div>
        
        <div class="management-section">
            <div class="form-section">
                <h2 class="section-title">添加/编辑歌曲</h2>
                <form id="songForm">
                    <input type="hidden" id="songId">
                    
                    <div class="form-group">
                        <label for="title">歌曲名称 *</label>
                        <input type="text" id="title" required>
                    </div>
                    
                    <div class="form-group">
                        <label for="author">作者</label>
                        <input type="text" id="author" placeholder="多个作者用逗号分隔">
                    </div>
                    
                    <div class="form-group">
                        <label for="band">所属乐队 *</label>
                        <select id="band" required>
                            <option value="">请选择乐队</option>
                            <option value="Ave Mujica">Ave Mujica</option>
                            <option value="MyGO!!!!!">MyGO!!!!!</option>
                            <!-- 可以动态添加更多乐队 -->
                        </select>
                    </div>
                    
                    <div class="form-group">
                        <label for="lyrics">歌词</label>
                        <textarea id="lyrics" placeholder="请输入歌词内容..."></textarea>
                    </div>
                    
                    <div class="btn-group">
                        <button type="submit" class="btn btn-primary">保存歌曲</button>
                        <button type="button" class="btn btn-secondary" onclick="resetForm()">重置</button>
                    </div>
                </form>
            </div>
            
            <div class="list-section">
                <h2 class="section-title">歌曲列表</h2>
                
                <div class="search-section">
                    <div class="search-form">
                        <input type="text" id="searchInput" placeholder="搜索歌曲名称...">
                        <button class="btn btn-primary" onclick="searchSongs()">搜索</button>
                    </div>
                    
                    <div class="filter-section">
                        <div class="filter-group">
                            <label for="bandFilter">按乐队筛选</label>
                            <select id="bandFilter" onchange="filterSongs()">
                                <option value="">所有乐队</option>
                                <option value="Ave Mujica">Ave Mujica</option>
                                <option value="MyGO!!!!!">MyGO!!!!!</option>
                            </select>
                        </div>
                    </div>
                </div>
                
                <div class="songs-list" id="songsList">
                    <!-- 歌曲列表将通过JavaScript动态生成 -->
                </div>
                
                <div class="pagination" id="pagination">
                    <!-- 分页控件将通过JavaScript动态生成 -->
                </div>
            </div>
        </div>
    </div>

    <script>
        let currentPage = 1;
        const pageSize = 5;
        let currentFilter = {
            band: '',
            title: ''
        };

        // 页面加载时获取歌曲列表和乐队信息
        document.addEventListener('DOMContentLoaded', function() {
            fetchBands();
            fetchSongs();
            setupForm();
        });

        // 设置表单提交事件
        function setupForm() {
            document.getElementById('songForm').addEventListener('submit', function(e) {
                e.preventDefault();
                saveSong();
            });
        }

        // 获取乐队列表
        async function fetchBands() {
            try {
                const response = await fetch('http://localhost:8000/api/bands');
                const bands = await response.json();
                populateBandSelects(bands);
            } catch (error) {
                console.error('获取乐队列表失败:', error);
            }
        }

        // 填充乐队选择框
        function populateBandSelects(bands) {
            const bandSelect = document.getElementById('band');
            const bandFilter = document.getElementById('bandFilter');
            
            // 清空现有选项（保留第一个选项）
            bandSelect.innerHTML = '<option value="">请选择乐队</option>';
            bandFilter.innerHTML = '<option value="">所有乐队</option>';
            
            bands.forEach(band => {
                const option1 = new Option(band.name, band.name);
                const option2 = new Option(band.name, band.name);
                bandSelect.add(option1);
                bandFilter.add(option2);
            });
        }

        // 获取歌曲列表
        async function fetchSongs(page = 1) {
            currentPage = page;
            try {
                let url = `http://localhost:8000/api/songs?page_index=${page}&page_size=${pageSize}`;
                
                if (currentFilter.band) {
                    url += `&band=${encodeURIComponent(currentFilter.band)}`;
                }
                if (currentFilter.title) {
                    url += `&title=${encodeURIComponent(currentFilter.title)}`;
                }
                
                const response = await fetch(url);
                const data = await response.json();
                displaySongs(data.songs || data);
                setupPagination(data.total || data.length);
            } catch (error) {
                console.error('获取歌曲列表失败:', error);
                showMessage('获取歌曲列表失败', 'error');
            }
        }

        // 显示歌曲列表
        function displaySongs(songs) {
            const container = document.getElementById('songsList');
            
            if (!songs || songs.length === 0) {
                container.innerHTML = '<div class="message">暂无歌曲数据</div>';
                return;
            }

            container.innerHTML = songs.map(song => `
                <div class="song-item">
                    <div class="song-header">
                        <div class="song-title">${song.title}</div>
                        <div class="song-band">${song.band}</div>
                    </div>
                    <div class="song-author">作者: ${song.author || '未知'}</div>
                    <div class="song-lyrics">${song.lyrics ? song.lyrics.substring(0, 100) + '...' : '暂无歌词'}</div>
                    <div class="song-actions">
                        <button class="action-btn btn-primary" onclick="editSong(${song.id})">编辑</button>
                        <button class="action-btn btn-danger" onclick="deleteSong(${song.id})">删除</button>
                    </div>
                </div>
            `).join('');
        }

        // 设置分页
        function setupPagination(total) {
            const totalPages = Math.ceil(total / pageSize);
            const pagination = document.getElementById('pagination');
            
            if (totalPages <= 1) {
                pagination.innerHTML = '';
                return;
            }
            
            let paginationHTML = '';
            
            // 上一页按钮
            if (currentPage > 1) {
                paginationHTML += `<button class="page-btn" onclick="fetchSongs(${currentPage - 1})">上一页</button>`;
            }
            
            // 页码按钮
            for (let i = 1; i <= totalPages; i++) {
                if (i === currentPage) {
                    paginationHTML += `<button class="page-btn active">${i}</button>`;
                } else {
                    paginationHTML += `<button class="page-btn" onclick="fetchSongs(${i})">${i}</button>`;
                }
            }
            
            // 下一页按钮
            if (currentPage < totalPages) {
                paginationHTML += `<button class="page-btn" onclick="fetchSongs(${currentPage + 1})">下一页</button>`;
            }
            
            pagination.innerHTML = paginationHTML;
        }

        // 保存歌曲（新增或更新）
        async function saveSong() {
            const songData = {
                title: document.getElementById('title').value,
                author: document.getElementById('author').value,
                lyrics: document.getElementById('lyrics').value,
                band: document.getElementById('band').value
            };

            const songId = document.getElementById('songId').value;

            try {
                let response;
                if (songId) {
                    // 更新歌曲
                    response = await fetch(`http://localhost:8000/api/songs/${songId}`, {
                        method: 'PUT',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify(songData)
                    });
                } else {
                    // 新增歌曲
                    response = await fetch('http://localhost:8000/api/songs', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify(songData)
                    });
                }

                if (response.ok) {
                    showMessage(`歌曲${songId ? '更新' : '添加'}成功`, 'success');
                    resetForm();
                    fetchSongs(currentPage);
                } else {
                    throw new Error('保存失败');
                }
            } catch (error) {
                console.error('保存歌曲失败:', error);
                showMessage('保存歌曲失败', 'error');
            }
        }

        // 编辑歌曲
        async function editSong(id) {
            try {
                const response = await fetch(`http://localhost:8000/api/songs/${id}`);
                const song = await response.json();
                
                document.getElementById('songId').value = song.id;
                document.getElementById('title').value = song.title || '';
                document.getElementById('author').value = song.author || '';
                document.getElementById('lyrics').value = song.lyrics || '';
                document.getElementById('band').value = song.band || '';
                
                showMessage('正在编辑歌曲，请修改后保存', 'success');
            } catch (error) {
                console.error('获取歌曲详情失败:', error);
                showMessage('获取歌曲详情失败', 'error');
            }
        }

        // 删除歌曲
        async function deleteSong(id) {
            if (!confirm('确定要删除这首歌曲吗？')) {
                return;
            }

            try {
                const response = await fetch(`http://localhost:8000/api/songs/${id}`, {
                    method: 'DELETE'
                });

                if (response.ok) {
                    showMessage('歌曲删除成功', 'success');
                    fetchSongs(currentPage);
                } else {
                    throw new Error('删除失败');
                }
            } catch (error) {
                console.error('删除歌曲失败:', error);
                showMessage('删除歌曲失败', 'error');
            }
        }

        // 搜索歌曲
        async function searchSongs() {
            const query = document.getElementById('searchInput').value.trim();
            currentFilter.title = query;
            currentPage = 1;
            fetchSongs(1);
        }

        // 筛选歌曲
        async function filterSongs() {
            const band = document.getElementById('bandFilter').value;
            currentFilter.band = band;
            currentPage = 1;
            fetchSongs(1);
        }

        // 重置表单
        function resetForm() {
            document.getElementById('songForm').reset();
            document.getElementById('songId').value = '';
        }

        // 显示消息
        function showMessage(text, type) {
            const messageDiv = document.getElementById('message');
            messageDiv.innerHTML = `<div class="message ${type}">${text}</div>`;
            
            setTimeout(() => {
                messageDiv.innerHTML = '';
            }, 3000);
        }
    </script>
</body>
</html>
```

### 1. **前端调用后端API流程**

- **用户交互与事件触发**

用户在前端提供的页面（网页等）进行操作，如提交表单，点击按钮等，前端代码能够捕获到这个操作事件（添加事件监听器:addEventListener()等）。

- **调用API请求函数**

前端代码调用一个封装好的API请求函数，例如band.html里的：

```javascript
async function fetchBandDetail(bandName) {
    try {
        const response = await fetch(`http://localhost:8000/api/bands?name=${encodeURIComponent(bandName)}`)
        // ...
    } catch (error) {...}
}
```

调用这个API请求函数

- **发送HTTP请求**

同样观察上面的代码，fetch()默认使用GET方法获取资源，`http://localhost:8000/api/bands` 是后端Router定义的API地址，`?`是分隔符，`name=${encodeURIComponent(bandName)}`参数通过URL查询字符串传递，后端Router需要从URL中解析处name。

- **后端Router接收请求**

这里的后端服务器运行在`http://localhost:8000`，接收到如`GET/api/bands?name=Poppin%27Party`(Poppin%27Party是编码后的bandName)这类的请求，然后找到对应的Cntroller/Handler函数，解析URL和所需操作，提取出`name = Poppin%27Party`和用户操作，再将提取出的参数和用户操作传递给Services层。

- **Services层处理业务逻辑**

接收Router传递来的参数，执行业务逻辑（用户操作），比如删除，添加等，这些操作需要由Router指定。

- **Services层返回结果**

将处理好的逻辑返回给Router层

- **Router构造并发送响应**

Router层将service层的数据封装成JSON字符串，并设置HTTP状态码，最后讲完整的HTTP响应（JSON数据和状态码）发送回前端

- **Models层定义数据结构，对request和respond进行过滤和验证**

设置数据结构，并用该数据结构解析前端发来的JSON请求体，验证字段是否合法。在后端返回数据是，还能把后端返回的数据格式化为设置的标准结构。

### **2. models层的设计：（简单来说就是提供一个request和respond的标准格式）**

```python
from pydantic import BaseModel #从导入BaseModel，这是所有pydantic模型的基类
from typing import Optional, List ## 导入类型提示工具，比如提示函数要传入什么类型的参数。List是列表，Dict是字典，Optional标识可能是指定类型或者None（可选）
from datetime import datetime # 用来生成时间戳

class BandBase(BaseModel):#继承BaseModel，Pydantic能够识别并赋予其作为数据模型的所有特殊能力和行为，例如数据验证能力。同时BaseModel提供了.model_dump(), .model_dump_json()等方法，能够将Pydantic模型转换为Python字典(.model_dump())和将python字典通过标准JSON编码器（如json.dumps())转换为一个JSON格式的字符串（.model_dump_json())
    name: str #乐队名称，必填字段. Pydantic要求输入必须能转换为一个非空字符串，这是数据验证的第一步
    description: str
    
class BandCreate(BandBase): #继承BandBase 表示创建乐队需要的格式
    pass # 占位符，表示仅继承，没有额外的字段

class BandResponse(BandBase): #表示返回给前端的数据模型
    id: int # 乐队的唯一标识符
    created_at: datetime #记录创建的时间戳
    class Config:
        from_attributes = True # 让Pydantic能从对象属性中读取数据，否则Pydantic智能从字典中读取

class SongBase(BaseModel):
    title: str
    author: Optional[str] = None #表示可以不提供此字段，默认是None。即若该字段缺失，则字段默认成None
    lyrics: Optional[str] = None
    band: str

class SongCreate(SongBase):
    pass

class SongUpdate(BaseModel):
    title: Optional[str] = None
    author: Optional[str] = None
    lyrics: Optional[str] = None
    band: Optional[str] = None

class SongResponse(SongBase):
    id: int
    created_at: datetime
    updated_at: datetime
    class Config:
        from_attributes = True

class PaginatedResponse(BaseModel):
    songs: List[SongResponse]
    total: int
    page_index: int
    page_size: int
```

### **3. 数据存储方式**

这里提供两种数据存储方式：文件存储和数据库存储

#### **(1) 文件存储需要的数据（放data文件夹下）：**

- band_info.json:

```json
[
  {
    "id": 1,
    "name": "MyGO!!!!!",
    "description": "迷失自我，但却向前。她们以充满情感的摇滚乐，表达年轻人的迷茫与坚定。",
    "created_at": "2025-10-21T00:05:15.094343"
  },
  {
    "id": 2,
    "name": "Ave Mujica",
    "description": "虚伪的假面，真实的自我。这是一个神秘且充满戏剧性的交响乐团，每个成员都带着面具。",
    "created_at": "2025-10-21T00:05:15.094357"
  },
  {
    "id": 3,
    "name": "Poppin'Party",
    "description": "充满活力的流行摇滚乐队，用音乐传递快乐和正能量。",
    "created_at": "2025-10-21T00:05:15.094358"
  },
  {
    "id": 4,
    "name": "Roselia",
    "description": "追求完美音乐表现的实力派乐队，以华丽的哥特式风格著称。",
    "created_at": "2025-10-21T00:05:15.094359"
  },
  {
    "id": 5,
    "name": "Afterglow",
    "description": "青梅竹马组成的硬摇滚乐队，音乐风格强烈而直接。",
    "created_at": "2025-10-21T00:05:15.094360"
  }
]
```

- song_data.json:

```json
[
  {
    "id": 1,
    "name": "MyGO!!!!!",
    "description": "迷失自我，但却向前。她们以充满情感的摇滚乐，表达年轻人的迷茫与坚定。",
    "created_at": "2025-10-21T00:05:15.094343"
  },
  {
    "id": 2,
    "name": "Ave Mujica",
    "description": "虚伪的假面，真实的自我。这是一个神秘且充满戏剧性的交响乐团，每个成员都带着面具。",
    "created_at": "2025-10-21T00:05:15.094357"
  },
  {
    "id": 3,
    "name": "Poppin'Party",
    "description": "充满活力的流行摇滚乐队，用音乐传递快乐和正能量。",
    "created_at": "2025-10-21T00:05:15.094358"
  },
  {
    "id": 4,
    "name": "Roselia",
    "description": "追求完美音乐表现的实力派乐队，以华丽的哥特式风格著称。",
    "created_at": "2025-10-21T00:05:15.094359"
  },
  {
    "id": 5,
    "name": "Afterglow",
    "description": "青梅竹马组成的硬摇滚乐队，音乐风格强烈而直接。",
    "created_at": "2025-10-21T00:05:15.094360"
  }
]
```

#### **(2) 数据库数据**

只用设置一个band.db在data文件夹下，初始是一个空数据库，并不自带示例数据。后续完成插入，更新，删除，查询操作，需要添加或更改数据库数据。

### **4. 后端service层和router层的搭建**

#### **(1) 文件存储的services层(root/backend/services/file_manager.py)**

```python
import json # python内部的json模块，处理JSON数据的编码和解码
import os # python内部的os模块，主要用来创建目录和检查文件是否存在
from typing import List, Dict, Optional
from datetime import datetime

class FileManager:
    def __init__(self): #构造函数，FileManager被创建时自动调用
        self.band_file = "data/band_info.json" #存储乐队信息的JSON文件路径
        self.song_file = "data/song_data.json"  #存储乐队信息的JSON文件路径
        self._ensure_data_files() # 下划线开头标识私有方法
        self._initialize_sample_data()
    
    def _ensure_data_files(self): #确保数据文件存在，如果不存在则创建空文件
        os.makedirs("data", exist_ok = True) #使用os模块创建名为“data”的目录， exist_ok=True表示如果目录已存在，不会抛出错误
        
        if not os.path.exists(self.band_file):# 若band_file不存在
            with open(self.band_file, 'w', encoding='utf-8') as f: #以写入模式打开乐队文件，若不存在则创建。使用utf-8支持中文
                json.dump([], f, ensure_ascii=False, indent=2) #将一个空列表[]写入文件，初始化为有效的JSON格式，ensure_ascii=False确保中文不被转义，indent=2将JSON执行缩进2的格式化
        if not os.path.exists(self.song_file):
            with open(self.song_file, 'w', encoding='utf-8') as f:
                json.dump([], f, ensure_ascii=False, indent=2)

    def _initialize_sample_data(self):# 初始化示例数据
        bands = self._read_bands() #读取乐队数据
        if not bands: #如果没有读到乐队数据
            sample_bands = [
                {
                    "id": 1,
                    "name": "MyGO!!!!!",
                    "description": "迷失自我，但却向前。她们以充满情感的摇滚乐，表达年轻人的迷茫与坚定。",
                    "created_at": datetime.now().isoformat()#取当前时间，将其转换为ISO 8601字符串格式
                },
                {
                    "id": 2,
                    "name": "Ave Mujica",
                    "description": "虚伪的假面，真实的自我。这是一个神秘且充满戏剧性的交响乐团，每个成员都带着面具。",
                    "created_at": datetime.now().isoformat()
                },
                {
                    "id": 3,
                    "name": "Poppin'Party",
                    "description": "充满活力的流行摇滚乐队，用音乐传递快乐和正能量。",
                    "created_at": datetime.now().isoformat()
                },
                {
                    "id": 4,
                    "name": "Roselia",
                    "description": "追求完美音乐表现的实力派乐队，以华丽的哥特式风格著称。",
                    "created_at": datetime.now().isoformat()
                },
                {
                    "id": 5,
                    "name": "Afterglow",
                    "description": "青梅竹马组成的硬摇滚乐队，音乐风格强烈而直接。",
                    "created_at": datetime.now().isoformat()
                }
            ]
            self._write_bands(sample_bands) #写入乐队数据
    def _read_bands(self) -> List[Dict]: #读取乐队数据 提示返回类型应该为字典列表
        try:
            with open(self.band_file, 'r', encoding='utf-8') as f: # 只读模式 若文件不存在，则会抛出FileNotFoundError异常
                bands = json.load(f) #读取JSON数据并解析为Python对象列表
                if not isinstance(bands, list): #若bands不是列表
                    return [] # 返回空列表
                return bands
        except (FileNotFoundError, json.JSONDecodeError): # 捕获文件未找到的错误或者JSON解析错误
            return []
    
    def _write_bands(self, bands: List[Dict]): #写入乐队数据 提示bands需要是一个字典列表
        with open(self.band_file, 'w', encoding='utf-8') as f:
            json.dump(bands, f, ensure_ascii=False, indent=2) #将字典列表转换为JSON数据写入bands中

    def _read_songs(self) -> List[Dict]:
        try:
            with open(self.song_file, 'r', encoding='utf-8') as f:
                songs = json.load(f)
                if not isinstance(songs, list):
                    return []
                return bands
        except(FileNotFoundError, json.JSONDecodeError):
            return []

    def _write_songs(self, songs: List[Dict]): # 写入歌曲数据
        with open(self.song_file, 'w', encoding='utf-8') as f:
            json.dump(songs, f, ensure_ascii=False, indent=2)
    
    def _generate_band_id(self) -> int: #生成乐队ID
        bands = self._read_bands()
        if not bands:
            return 1
        return max(band.get('id', 0) for band in bands) + 1 #0是默认值，若没有找到id，则返回默认值+1，用于处理无id情况。若有最大id，则+1返回一个新id，保证id逐个递增

    def _generate_song_id(self) -> int: #生成歌曲ID
        songs = self._read_songs()
        if not songs:
            return 1
        return max(song.get('id', 0) for song in songs) + 1

    #乐队相关操作
    def get_all_bands(self) -> List[Dict]:#获取所有乐队
        return self._read_bands()
    
    def get_band_by_name(self, name: str) -> Optional[Dict]: #用名字找乐队（Optional提示若没找到乐队则可以返回None）
        bands = self._read_bands()
        for band in bands:
            if band.get('name') == name:
                return band
        return None
    
    def get_band_by_id(self, id: int) -> Optional[Dict]:# 用id找乐队
        bands = self._read_bands()
        for band in bands:
            if band.get('id') == name:
                return band
        return None

    def create_band(self, band_data: Dict) -> Dict: # 创建乐队
        bands = self._read_bands()
        #检查乐队名称是否存在
        if any(band.get('name') == band_data.get('name') for band in bands):
            raise ValueError("乐队名称已存在") #主动触发异常
        #生成新id和时间段
        band_data['id'] = self._generate_band_id()
        band_data['created_at'] = datetime.now().isoformat()

        bands.append(band_data)
        self._write_bands(bands)
        return band_data

    #歌曲相关操作
    def get_all_songs(self) -> List[Dict]:#获取所有歌曲
        return self._read_songs()
    
    def get_songs_by_id(self, id: int) -> Optional[Dict]:# 用id找歌
        songs = self._read_songs()
        for song in songs:
            if song.get('id') == name:
                return song
        return None

    def get_songs_by_band(self, band_name: str) -> List[Dict]: # 用乐队名找歌
        songs = self._read_songs()
        return [song for song in songs if song.get("band") == band_name]

    def search_songs_by_title(self, title: str) -> List[Dict]: # 用标题找歌
        songs = self._read_songs()
        title_lower = title.lower()
        return [song for song in songs if title_lower in song.get("title", "").lower()] #用[]隐式创建一个空列表，若False则返回空列表；""表示默认值为空，防止song缺失title键

    def create_song(self, song_data: Dict) -> Dict: # 创建歌曲
        #验证乐队是否存在
        band = self.get_band_by_name(song_data.get("band"))
        if not band:
            raise ValueError("所属乐队不存在")
        songs = self._read_songs()
        if any(song.get('name') == song_data.get('title') for song in songs):
            raise ValueError("歌曲名称已存在") #主动触发异常
        #生成新id和时间段
        song_data['id'] = self._generate_song_id()
        song_data['created_at'] = datetime.now().isoformat()
        song_data["updated_at"] = song_data["created_at"]

        songs.append(song_data)
        self._write_songs(songs)
        return song_data 
    
    def update_song(self, song_id: int, song_data: Dict) -> Optional[Dict]:# 更新歌曲信息
        songs = self._read_songs()
        for song in songs:
            if song.get('id') == song_id:
                band_name = song_data.get("band")
                if band_name:
                    bands = self._read_bands()
                    if not any(band.get("name") == band_name for band in bands):
                        return None
                #更新字段
                for key in ["title", "author", "lyrics", "band"]: #来源于Models
                    if key in song_data:
                        song[key] = song_data[key]
                song["updated_at"] = datatime.now().isoformat()
                
                self._write_songs(songs)
                return song
        return None
    
    def delete_song(self, song_id: int) -> bool: #删除歌曲
        songs = self._read_songs()
        new_songs = [song for song in songs if song.get("id") != song_id]
        if len(new_songs) == len(songs):
            return False
        self._write_songs(new_songs)
        return True
```

#### **(2) 文件存储的routers层(root/backend/routers/band_with_file.py)**

为了更好理解下面的代码，先对FastAPI做一点解释

FastAPI中的参数主要有三种主要来源：

- 查询参数：URL中类似于的`?name=xxx`这类的，对应函数是Query(). ex: /api/bands?name=Roselia

- 路径参数：URL的一部分，对应函数是Path(). ex: /api/songs/5 这里/api/songs是固定路径，5就是路径参数，会被绑定在song_id上

- 请求体：POST/PUT发送的JSON数据，对应函数是Body(). ex: {"title": "new song"}

```python
from fastapi import APIRouter, HTTPException, Query, Path # 从fastapi导入核心组建：APIRouter: 用于组织和管理路由类；HTTPException: 用于在请求处理中抛出HTTP错误；Query: 定义查询参数的验证规；Path：定义路径参数的验证规则
from typing import Optional, List
from services.file_manager import FileManager # 导入services层的FileManager类
from models.bangdream_models import (
    BandResponse, SongCreate, SongResponse, SongUpdate, PaginatedResponse
) # 导入Models层的Pydantic模型

router = APIRouter(prefix="/api", tag["文件存储版本"]) #创建一个APIRouter实例，主要目的是不用重复添加前缀和标签，还能独立写多个独立的接口文件（用数据库存储的routers层），最后在main.py里统一import
file_manager = FileManager() #创建一个类

@router.get("/bands", response_model=List[BandResponse]) #添加一个路由装饰器，将下面的函数注册为GET/api/bands的接口，并指定返回的响应模型为models层提供的BandResponse列表
def get_bands(name: Optional[str] = Query(None, description="乐队名称")): #让FastAPI明确如何从HTTP中提取参数，同时可以添加一些验证逻辑（比如name中必须有什么什么这类的，这里没有添加）和描述（description，目的是让代码易读）。不加Query的话，FastAPI也能自动推理，但是就无法添加验证逻辑和描述
    """获取乐队列表或根据名称查询特定乐队"""
    if name:
        band = file_manager.get_band_by_name(name)
        if band:
            return [band]
        else:
            raise HTTPException(status_code=404, detail="乐队不存在") #返回HTTP错误响应 错误码为404
     return file_manager.get_all_bands()

@router.get("/songs", response_model=PaginatedResponse) #GET默认状态码为200 OK
def get_songs(
    band: Optional[str] = Query(None, description="乐队名称")
    title: Optional[str] = Query(None, description="歌曲名称")
    page_index: int = Query(1, ge=1, description="页码") # 这里添加了默认值和验证逻辑，默认值是1，验证逻辑ge：大于等于；验证逻辑le：小于等于
    page_size: int = Query(10, ge=1, le=100, description="每页数量")
):
    all_songs = file_manager._read_songs()
    filtered_songs = all_songs
    if band:
        filtered_songs = [song for song in filtered_songs if song.get("band") == band]
    if title:
        filtered_songs = [song for song in filtered_songs if title.lower() in song.get("title", "").lower()]
    
    total = len(filtered_songs) # 记录总页数
    start = (page_index - 1) * page_size #分页逻辑：每页显示page_size页，用户请求第page_index页，开始就是第(page_index - 1) * page_size页。比如：请求第2页，每页显示3首哥，则返回第4，5，6首歌，则从需要从下标3开始
    end = start + page_size
    paginated_songs = filtered_songs[start:end] #[start:end]start取，end不取

    return {
        "songs": paginated_songs,
        "total": total,
        "page_index": page_index,
        "page_size": page_size
    }

@router.get("/songs/{song_id}", response_model=SongResponse)    
def get_song_detail(song_id: int = Path(..., ge=1, description="歌曲ID")):# 这里的song_id为必填项，用...代替
    song = file_magager.get_song_by_id(song_id)
    if not song:
        raise HTTPException(status_code=404, detail="歌曲不存在")
    return song

@router.post("/songs", response_model=SongResponse, status_code=201)
def create_song(song: SongCreate):
    try:
        return file_manager.create_song(song.model_dump())
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))

@router.put("/songs/{song_id}", response_model=SongResponse)
def update_song(song_id: int = Path(..., ge=1, description="歌曲ID"), song:SongUpdate=None):
    try:
        updated = file_manager.update_song(song_id, song.model_dump(exclude_unset=True)) #只更新传入的字段，若title这些是None，则不传入
        if not updated:
            raise HTTPException(status_code=404, detail="歌曲不存在")
        return updated
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))

@router.delete("/songs/{song_id}", status_code=204)
def delete_song(song_id: int = Path(..., ge=1, description="歌曲ID")):
    deleted = file_manager.delete_song(song_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="歌曲不存在")
```

这里有状态码的写入，做如下解释:

FastAPI的默认状态码是200 OK（状态码由状态码数字和状态文本组成）。
HTTP方法：

- **GET**：语义：从服务器获取资源。标准状态码就是200 OK，说明请求成功并返回数据；
- **POST时**：语义：往服务器发送数据。标准状态码是201 Created（状态文本自动创建），说明成功创建资源；
- **PUT**：语义：用新数据替换旧资源。标准状态码是200 OK或者204 No Content（状态文本自动创建），说明成功更新资源；
- **DELETE**：语义：删除服务器上的资源。标准状态码是204 No Content（状态文本自动创建），说明成功删除。

#### **(3) 数据库存储的service层**

为更好理解数据库版本，这先提供一些数据库的基础交互操作：

- **打开数据库：**

    这里用sqlite数据库。进入项目根目录，然后用在终端输入sqlite3 data/band.db
`cd your_project_root`
`sqlite3 data/band.db`

- **创建一张测试表：**

```sql
CREATE TABLE IF NOT EXISTS test_users (
    id INTGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER
);
```

- **插入单条数据**

```sql
INSERT INTO test_users(name, age) VALUES('Alice', 22);
```

- **查询表中所有记录**

```sql
SELECT * FROM test_users;
```

- **更新一条记录**

```sql
UPDATE test_users SET age = 23 WHERE name = 'Alice';
```

- **退出**

```sql
.exit
```

```python
import sqlite3 #引入sqlite3数据库
import os #用于文件/目录操作
from typing import List, Optional, Tuple, Dict, Any

class DatabaseManager:
    def __init__(self, db_path: str = "data/band.db"):
        self.db_path = db_path
        self.init_database()
    
    def get_connection(self):# 获取数据库连接
        conn = sqlite3.connect(self.db_path) #打开/创建数据库文件并返回链接对象conn
        conn.row_factory = sqlite3.Row #将查询返回的行类型设置为sqlite3.Row，这样每次取出的结果都是sqlite3.Row这种类型而不是默认的Tuple，用于后续的row_to_dict最后将row变成dict，就可以用dict(row)取row
        return conn

    def init_database(self):
        os.makedirs(os.path.dirname(self.db_path), exist_ok=True) #确保data目录存在，没有则创建
        conn = self.get_connection()
        try:
            cursor = conn.cursor()#创建游标,作为数据库操作的中间人，负责执行SQL并获取结果，
             # 创建乐队表
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS bands (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT UNIQUE NOT NULL,
                    description TEXT NOT NULL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
            
            # 创建歌曲表
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS songs (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT NOT NULL,
                    author TEXT,
                    lyrics TEXT,
                    band TEXT NOT NULL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')

            # 插入初始乐队数据
            initial_bands = [
                ("MyGO!!!!!", "迷失自我，但却向前。她们以充满情感的摇滚乐，表达年轻人的迷茫与坚定。"),
                ("Ave Mujica", "虚伪的假面，真实的自我。这是一个神秘且充满戏剧性的交响乐团，每个成员都带着面具。"),
                ("Morfonica", "如梦似幻的交响乐团。她们以小提琴为主轴，演奏出优雅而华丽的乐章。")
            ]

            #批量插入
            cursor.executemany(
                "INSERT OR IGNORE INTO bands (name, description) VALUES (?, ?)", # IGNORE表示，已存在就忽略插入
                initial_bands
            )

            #检查歌曲表是否为空
            cursor.execute("SELECT COUNT(*) as count FROM songs") # COUNT(*)表示统计表中所有行的数量（包括空字段行），并给统计结果取一个别名count
            song_count = cursor.fetchone()["count"] # cursor.fetchone()获取查询结果的第一行，由于是获取数量，则查询结果只有一行：{"count": 5}

            if song_count == 0:
                if song_count == 0:
            # 插入一些示例歌曲数据
                initial_songs = [
                    ("黑色生日", "Doloris", "歌词内容...", "Ave Mujica"),
                    ("迷星叫", "MyGO!!!!!", "歌词内容...", "MyGO!!!!!")
                ]
                
                cursor.executemany(
                    "INSERT OR IGNORE INTO songs (title, author, lyrics, band) VALUES (?, ?, ?, ?)",
                    initial_songs
                )
            
            conn.commit() #提交到数据库

        finally:
            conn.close() #关闭连接
    
    #将sqlite3.Row转换为字典
    def row_to_dict(self, row) -> Dict[str, Any]: #Any表示可以是任何值
        if row is None:
            return None
        return dict(row)

    # 乐队相关操作
    def get_all_bands(self, name: str) -> List[Dict[str, Any]]:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM bands ORDER BY name")
            rows = cursor.fetchall()
            return [self.row_to_dict(row) for row in rows]
        finally:
            conn.close()
    
    def get_band_by_name(self, name: str) -> Optional[Dict[str, Any]]:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            cursor = execute("SELECT * FROM bands WHERE name = ?", (name,)) #execute方法要求第二个参数是一个元组或者列表，因此要用(name,)表示只有一个元素的元组
            row = cursor.fetchone() #取一行结果
            return self.row_to_dict(row)
        finally:
            conn.close()

    # 歌曲相关操作
    def get_songs(
        self,
        band: Optional[str] = None,
        title: Optional[str] = None,
        page_index: int = 1,
        page_size: int = 10
    ) -> Tuple[List[Dict[str, Any]], int]:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            query = "SELECT * FROM songs WHERE 1=1" #1=1永远TRUE，占位，后续澄清他的作用
            params = [] #收集band和title

            if band:
                query += " AND band = ?" # 用1=1的原因就在这里，这样query就可以变成"SELECT * FROM songs WHERE 1=1 AND band = ?"这样就不用再写一个ifelse让第一个不加AND。这样子第一个加AND也不会出错。问号是占位符，后续用params里的值替换
                params.append(band)
            if title:
                query += "AND title LIKE = ?"
                params.append(f"%{title}%") #这里LIKE是模糊查询，只要包含title就算，因此这里要将title用%%包围起来，做模糊查询
            
            count_query = f"SELECT COUNT(*) FROM ({query})"
            cursor.execute(count_query, params)  
            total = cursor.fetchone()[0] #fetchone()会返回一个元组，类似于(43,)因此要取index=0的值

            query += " ORDER BY created_at DESC LIMIT ? OFFSET ?" #从按created_at降序（DESC）排序后的结果中，跳过前(page_index - 1) * page_size条记录（OFFSET），然后取出接下来page_size（LIMIT）条
            params.extend([page_size, (page_index - 1) * page_size])
            cursor.execute(query, params)
            rows = cursor.fetchall()

            return [self.row_to_dict(row) for row in rows], total
        finally:
            conn.close()
        
    def get_song_by_id(self, song_id: int) -> Optional[Dict[str, Any]]:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM songs WHERE id = ?", (song_id,))
            row = cursor.fetchone
            return self.row_to_dict(row)
        finally:
            conn.close()
    
    def create_song(self, song_data: dict) -> int:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            cursor.execute(
                "INSERT INTO songs (title, author, lyrics, band) VLALUES (?, ?, ?, ?)",
                (
                    song_data["title"],
                    song_data.get("author"),
                    song_data.get("lyrics"),
                    song_data["band"],
                ),
            )
            conn.commit()
            return cursor.lastrowid # lastrowid -> last row id可以拿到新建的这个歌的id
    finally:
        conn.close()

    def update_song(self, song_id: int, song_data: dict) -> bool:
        conn = self.get_connction()
        try:
            cursor = conn.cursor()
            cursor.execute(
                """
                UPDATE songs SET title = ?, author = ?, lyrics = ?, band = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?
                """,
                (
                    song_data["title"],
                    song_data.get("author"),
                    song_data.get("lyrics"),
                    song_data["band"],
                    song_id,
                ),
            )
            conn.commit()
            return cursor.rowcount > 0
        finally:
            conn.close()

    def delete_song(self, song_id: int) -> bool:
        conn = self.get_connection()
        try:
            cursor = conn.cursor()
            cursor.execute("DELETE FROM songs WHERE id = ?", (song_id,))
            conn.commit()
            return cursor.rowcount > 0 # 注意这里的rowcount检测的是SQL语句执行后被影响的行数
        finally:
            conn.close()
```

#### **(4) 数据库存储的router层(root/backend/services/db_manager.py)**

```python
from fastapi import APIRouter, HTTPException, Query
from typing import Optional, List
from models.bangdream_models import (
    BandResponse, SongCreate, SongResponse, SongUpdate, PaginatedResponse
)
from service.db_manager import DatabaseManager

router = APIRouter(prefix="/api", tags=["bands"])
db_manager = DatabaseManager()

@router.get("/bands", response_model=List[BandResponse])
def get_bands(name: Optional[str] = Query(None)):
    """获取所有乐队或按名称查询特定乐队"""
    if name:
        band = db_manager.get_band_by_name(name)
        if not band:
            raise HTTPException(status_code=404, detail="乐队不存在")
        return [band]
    return db_manager.get_all_bands()

@router.get("/songs", response_model=PaginatedResponse) 
def get_songs(
    band: Optional[str] = Query(None),
    title: Optional[str] = Query(None),
    page_index: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100)
): # 这里的数据库写法实际上在services层就完成了过滤和分页逻辑
    songs, total = db_manager.get_songs(band, title, page_index, page_size)
    return {"songs": songs, "total": total, "page_index": page_index, "page_size": page_size} 

@router.post("/songs", response_model=SongResponse, status_code=201)
def create_song(song: SongCreate): # 这里要判断而文件写法不判断是因为文件写法在services层判断过了
    band = db.manager.get_band_by_name(song.band)
    if not band:
       raise HTTPException(status_code=404, detail="乐队不存在")
    song_id = db_manager.create_song(song.model_dump())
    return db_manager.get_song_by_id(song_id)

@router.put("/songs/{song_id}", response_model=SongResponse)
def update_song(song_id: int, song: SongUpdate):
    existing_song = db_manager.get_song_by_id(song_id)
    if not existing_song:
        raise HTTPException(status_code=404, detail="歌曲不存在")
    if song.band:
        band = db_manager.get_band_by_name(song.band)
        if not band:
            raise HTTPException(status_code=404, detail="乐队不存在")
    success = db_manager.update_song(song_id, song.model_dump())
    if not success:
        raise HTTPException(status_code=404, detail="更新失败，歌曲不存在")
    return db_manager.get_song_by_id(song_id)

@router.delete("/songs/{song_id}", status_code=204)
def delete_song(song_id: int):
    """删除歌曲"""
    success = db_manager.delete_song(song_id)
    if not success:
        raise HTTPException(status_code=404, detail="删除失败，歌曲不存在")
```

#### **(5) 编写一个自动化脚本向数据库里插入随机生成的数据**

```python
import sqlite3
import random
from datetime import datetime

DB_PATH = "data/band.db"

def insert_random_songs(n = 100):
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.cursor()
    bands = ["MyGO!!!!!", "Ave MUjica", "Morfonica"]
    for i in range(n):
        title = f"测试歌曲_{i}"
        author = ramdom.choice(["Doloris", "Anon", "Shizuku"])
        lyrics = f"这是测试歌词{i}..."
        band = random.choice(bands)
        cursor.execute("""
            INSERT INTO songs (title, author, lyrics, band, created_at, updated_at) VALUES(?, ?, ?, ?, ?, ?)""", (title, author, lyrics, band, datetime.now(), datetime.now()))
        
    conn.commit()
    conn.close()
    print(f"成功插入{n}条随机歌曲数据")

if __name__ == "__main__": # Python程序入口判断语句，作用是判断当前文件是被直接运行还是被导入到其他文件中。"__main__"意思为当直接运行这个文件才为TRUE，不能从其他文件导入这个模块
    insert_random_songs(100) 
```

### **5. main.py后端入口搭建**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware # 导入跨域中间件，解除前端网页（运行在localhost:5500）访问后端（运行在local:8000)的安全限制
import os

USE_FILE_STORAGE = True # 设定一个bool作为开关，用来控制用哪一种存储方式

app = FastAPI( # 这是一个FastAPI应用对象
    title="BanG Dream! 乐队管理系统", # API文档标题
    description="基于FastAPI的乐队和歌曲管理API" + (" (文件存储版本) " if USE_FILE_STORAGE else " (数据库版本) "), # API文档描述
    version="1.0.0" # API 版本号
)

app.add_middleware( # 配置跨域中间件
    CORSMiddleware,
    allow_origins=["*"], # 允许所有来源（前端网站）访问
    allow_credentials=True, # 允许携带cookies
    allow_methods=["*"], # 允许所有HTTP方法
    allow_headers=["*"] # 允许所有请求头
)

if USE_FILE_STORAGE:
    from routers.band_with_file import router as band_router
    print("使用文件存储版本")
else:
    from routers.band_with_db import router as band_router
    print("使用数据库版本")

app.include_router(band_router) #将路由挂载到主应用上,也就是将定义好的所有接口，注册到主应用app里（app就是上面FastAPI应用对象）这样当前端访问http://localhost:8000/api/songs之类的时候，就能将请求转发给band_with_file.py 里的 get_songs()。

@app.get("/")
async def root(): #表示这是一个HTTP GET接口，路径是根目录，访问http://localhost:8000/会返回下面内容
    return {"message": "BanG Dream! 乐队管理系统API", "storage_type": "file" if USE_FILE_STORAGE else "database"}

@app.get("/health") #健康检查接口，用于确认服务器正常运行
async def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn #这是一个轻量级的Python Web服务器，可以监听http://localhost:8000。这样使浏览器，postman和前端页面都能访问后端接口
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### **6. sync和async的简易比较并增加异步获取歌曲信息的方法**

#### **(1) sync和async的简易比较**

模拟同时向十个手机号发送验证短信

- **sync:**

```python
import time
import random

def send_verification_code(phone_number): # 发送验证码函数
    code = random.randint(100000, 999999)
    print(f"Sending verification code {code} to {phone_number}")
    time.sleep(1) # 模拟一秒的网络延迟（同步阻塞）
    return code

def main(phone_numbers):
    results = []
    for phone in phone_numbers:
        code = send_verification_code(phone)
        results.append(code)
    return results

if __name__ == "__main__":
    phone_numbers = [f"138000000{i}" for i in range(10)] #十个测试手机号
    start_time = time.time()
    codes = main(phone_numbers)
    end_time = time.time()
    print("\nVerification codes:", codes)
    print(f"Total time taken: {end_time - start_time:.2f} seconds")
```

十个手机号，每个手机号需要等待1s，则一共需要等待10s

- **async:**

```python
import asyncio
import time
import random

def send_verification_code(phone_number): # 发送验证码函数
    code = random.randint(100000, 999999)
    print(f"Sending verification code {code} to {phone_number}")
    await asyncio.sleep(1) # 异步等待（非阻塞）
    return code

def main(phone_numbers):
    tasks = [send_verification_code(phone) for phone in phone_numbers] # 存储所有处于异步等待的函数
    results = await asyncio.gather(*tasks) # 并发执行所有任务并收集结果，由于异步等待的所有操作一起执行，故事件由里面最慢的决定，约为1s
    return results

if __name__ == "__main__":
    phone_numbers = [f"138000000{i}" for i in range(10)] #十个测试手机号
    start_time = time.time()
    codes = asyncio.run(main(phone_numbers)) #这样显式运行才能启动异步函数
    end_time = time.time()
    print("\nVerification codes:", codes)
    print(f"Total time taken: {end_time - start_time:.2f} seconds")
```

结论：需要在I/O密集型且实际大量并发连接或长时间等待的场景中使用异步编程

#### **(2) 添加异步获取歌曲信息的方法**

- **在services中的db_manager.py添加异步业务逻辑**

```python
    import aiosqlite
    ...
    class DatabaseManager:
        ...
        async def async_get_songs(
            self,
            band: Optional[str] = None,
            title: Optional[str] = None,
            page_index: int = 1,
            page_size: int = 10
        ) -> Tuple[List[Dict[str, Any]], int]:
            async with aiosqlite.connect(self.db_path) as conn: # 建立数据库连接是一个I/O密集型操作，涉及文件系统访问和初始化资源
                conn.row_factory = sqlite3.Row
            query = "SELECT * FROM songs WHERE 1=1"
            params = []

            if band:
                query += " AND band = ?"
                params.append(band)
            if title:
                query += " AND title LIKE ?"
                params.append(f"%{title}%")

            count_query = f"SELECT COUNT(*) FROM ({query})"
            async with conn.execute(count_query, params) as cursor: # 数据库操作需要等待磁盘I/O操作完成
                total = (await cursor.fetchone())[0] #从数据库获取数据仍然是一个I/O阻塞性操作
            
            query += " ORDER BY created_at DESC LIMIT ? OFFSET ?"
            params.extend([page_size, (page_index - 1) * page_size])
            async with conn.execute(query, params) as cursor: #数据库读写是I/O密集型操作
                rows = await cursor.fetchall() #获取所有结果可能涉及大量数据，是一个长时间的I/O等待
            
            return [self.row_to_dict(row) for row in rows], total
```

- **在routers中的band_with_db.py添加异步业务逻辑**

```python
@router.get("/songs", response_model=PaginatedResponse)
async def async_get_songs(
    band: Optional[str] = Query(None),
    title: Optional[str] = Query(None),
    page_index: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100)
):
    songs, total = await db_manager.get_songs(band, title, page_index, page_size) # 使用的方法是一个异步函数，返回一个协程对象，await告诉事件循环开始执行这个协程对象，等待其完成并获取最终结果
    return {"songs": songs, "total": total, "page_index": page_index, "page_size": page_size}
```

#### **(3) 编写压力测试**

```python
import asyncio
import time
import requests

URL_SYNC = "http://localhost:8000/api/songs"
URL_ASYNC = "http://localhost:8000/api/songs/async"

async def fetch_async(url, page):
    await asyncio.to_thread(requests.get(f"{url}?page_index={page}&page_size=5")) # requests.get()是一个同步阻塞函数，与asyncio的工作机制并不兼容，因此需要用asyncio.to_thread()这个工具来解决同步函数阻塞的问题。他的原理是将传入的同步函数及其参数提交给一个独立的线程池去执行，返回一个awaitable可等待对象，实现异步。
    
async def run_async_load(url, n=100):
    tasks = [fetch_async(url, i % 10 + 1) for i in range(n)]
    await asyncio.gather(*tasks)

def run_sync_load(url, n=100):
    for i in range(n):
        request.get(f"{url}?page_index={page}&page_size=5"))

if __name__ == "main__":
    print("Running sync test...")
    start = time.time()
    run_sync_load(URL_SYNC, 100)
    print(f"Sync total time: {time:time() - start:.2f}s")

    print("Running async test...")
    start = time.time()
    asyncio.run(run_async_load(URL_ASYNC, 100))
    print(f"Async total time: {time.time() - start:.2f}s")
```

### **7. postman测试的基本思想**

参考Postman使用教程[Postman使用教程](https://www.cnblogs.com/-lhl/articles/18663600).

### **8. 解决环境依赖**

配置虚拟环境：

- 在项目根目录下运行 `python3 -m venv venv`

- 激活虚拟环境：
    MacOS/Linux: `source venv/bin/activate`
    Windows(CMD): `venv\Scripts\activate`
    Windows(PowerShell): `.\venv\Scripts\Activate.ps1`

- 在用命令行安装依赖：`pip install -r requirements.txt`

下面是 `requirements.txt` 的内容（放在根目录下）：

aiohappyeyeballs==2.6.1
aiohttp==3.13.1
aiosignal==1.4.0
aiosqlite==0.21.0
annotated-types==0.7.0
anyio==4.11.0
async-timeout==5.0.1
attrs==25.4.0
certifi==2025.10.5
charset-normalizer==3.4.4
click==8.3.0
exceptiongroup==1.3.0
fastapi==0.119.0
frozenlist==1.8.0
h11==0.16.0
idna==3.11
multidict==6.7.0
propcache==0.4.1
pydantic==2.12.3
pydantic_core==2.41.4
requests==2.32.5
sniffio==1.3.1
starlette==0.48.0
typing-inspection==0.4.2
typing_extensions==4.15.0
urllib3==2.5.0
uvicorn==0.38.0
yarl==1.22.0
