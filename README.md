<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UID Data Processor</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f8f9fa;
            color: #343a40;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 1000px;
            margin: auto;
            padding: 20px;
            background: #ffffff;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            margin-top: 40px;
        }
        h1, h2 {
            text-align: center;
            color: #ff6600;
        }
        .button, .input-file {
            display: inline-block;
            padding: 10px 20px;
            margin: 10px 0;
            font-size: 16px;
            color: #ffffff;
            background: linear-gradient(90deg, #ff6600, #ff3300);
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.3s ease;
            text-align: center;
            text-decoration: none;
        }
        .button:hover, .input-file:hover {
            background: linear-gradient(90deg, #ff3300, #ff6600);
        }
        .section {
            margin: 20px 0;
        }
        .file-info, .file-preview {
            margin: 20px 0;
            padding: 10px;
            background-color: #e9ecef;
            border-radius: 4px;
            overflow: auto;
            max-height: 200px;
        }
        .file-preview pre {
            white-space: pre-wrap;
            word-wrap: break-word;
        }
        .footer {
            text-align: center;
            padding: 10px 0;
            margin-top: 40px;
            font-size: 14px;
            color: #ffffff;
            background-color: #ff6600;
            border-top: 2px solid #ff3300;
        }
        #progress {
            display: none;
            text-align: center;
            margin: 20px 0;
            font-size: 18px;
            color: #dc3545;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        #chart {
            margin: 20px 0;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <div class="container">
        <h1>UID Data Processor</h1>
        
        <div class="section">
            <h2>Tệp bạn muốn lọc</h2>
            <input type="file" id="fileInput" class="input-file">
            <div id="fileInfo"></div>
            <div id="filePreview" class="file-preview"></div>
        </div>
        
        <div class="section">
            <h2>Tệp so sánh</h2>
            <input type="file" id="newFileInput" class="input-file">
            <div id="newFileInfo"></div>
            <div id="newFilePreview" class="file-preview"></div>
        </div>
        
        <button class="button" onclick="filterDuplicates()">Filter Duplicates</button>
        
        <div id="progress">Processing...</div>
        
        <div id="finalReport" style="display: none;"></div>
        
        <button id="downloadLink" class="button" style="display:none;" onclick="downloadFile()">Download Processed File</button>
        <button class="button" onclick="copyToClipboard()" style="display:none;">Copy to Clipboard</button>
        
        <div class="section">
            <h2>Nhập dữ liệu</h2>
            <textarea id="inputData" rows="10" style="width: 100%;"></textarea>
            <button class="button" onclick="processData()">Xử lý dữ liệu</button>
        </div>

        <div class="section" id="resultSection" style="display:none;">
            <h2>Kết quả xử lý</h2>
            <div id="stats"></div>
            <table id="resultTable">
                <thead>
                    <tr>
                        <th>STT</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>ID</th>
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
            <button class="button" onclick="downloadExcel()">Download Excel</button>
            <button class="button" onclick="copyToClipboard()">Copy to Clipboard</button>
        </div>
        
        <div class="section" id="chartSection" style="display:none;">
            <h2>Biểu đồ</h2>
            <canvas id="chart"></canvas>
        </div>
    </div>
    <div class="footer">
        Copyright © 2024 Ken Chen, (+84) 964 266 706
    </div>

    <script>
        let processedUidList = [];

        function processFile(event) {
            const file = event.target.files[0];
            const reader = new FileReader();
            reader.onload = function(e) {
                const content = e.target.result;
                document.getElementById('filePreview').innerText = content;
                document.getElementById('fileInfo').innerText = `File Name: ${file.name}, File Size: ${file.size} bytes`;
                processUidList(content, 'file');
            };
            reader.readAsText(file);
        }

        function uploadNewFile(event) {
            const file = event.target.files[0];
            const reader = new FileReader();
            reader.onload = function(e) {
                const content = e.target.result;
                document.getElementById('newFilePreview').innerText = content;
                document.getElementById('newFileInfo').innerText = `File Name: ${file.name}, File Size: ${file.size} bytes`;
                processUidList(content, 'newFile');
            };
            reader.readAsText(file);
        }

        function processUidList(content, type) {
            const lines = content.split(/\r?\n/);
            let uidList = [];
            lines.forEach(line => {
                const uid = line.trim();
                if (uid) uidList.push(uid);
            });

            if (type === 'file') {
                processedUidList = uidList;
            } else {
                compareUidList(uidList);
            }
        }

        function compareUidList(newUidList) {
            const initialLength = processedUidList.length;
            const newInitialLength = newUidList.length;

            processedUidList = processedUidList.filter(uid => !newUidList.includes(uid));
            newUidList = newUidList.filter(uid => !processedUidList.includes(uid));

            const finalLength = processedUidList.length;
            const newFinalLength = newUidList.length;
            const removedCount = initialLength - finalLength;
            const newRemovedCount = newInitialLength - newFinalLength;

            const duplicatePercentage = ((removedCount / initialLength) * 100).toFixed(2);
            const newDuplicatePercentage = ((newRemovedCount / newInitialLength) * 100).toFixed(2);

            document.getElementById('finalReport').innerText = `
                Old File: Initial Length: ${initialLength}, Final Length: ${finalLength}, Duplicates removed: ${removedCount}, Duplicate percentage: ${duplicatePercentage}%\n
                New File: Initial Length: ${newInitialLength}, Final Length: ${newFinalLength}, Duplicates removed: ${newRemovedCount}, Duplicate percentage: ${newDuplicatePercentage}%
            `;
            document.getElementById('finalReport').style.display = 'block';
            document.getElementById('downloadLink').style.display = 'inline-block';
            document.querySelector('.button[onclick="copyToClipboard()"]').style.display = 'inline-block';
        }

        function downloadFile() {
            const blob = new Blob([processedUidList.join('\n')], { type: 'text/plain' });
            const url = URL.createObjectURL(blob);
            const link = document.getElementById('downloadLink');
            link.href = url;
            link.download = 'processed_uids.txt';
            link.click();
            URL.revokeObjectURL(url);
        }

        function copyToClipboard() {
            const text = processedUidList.join('\n');
            navigator.clipboard.writeText(text).then(() => {
                alert("Processed UIDs copied to clipboard!");
            }, (err) => {
                console.error('Failed to copy: ', err);
            });
        }

        document.getElementById('fileInput').addEventListener('change', processFile);
        document.getElementById('newFileInput').addEventListener('change', uploadNewFile);

        function filterDuplicates() {
            document.getElementById('progress').style.display = 'block';
            setTimeout(() => {
                document.getElementById('progress').style.display = 'none';
                compareUidList([]);
            }, 2000); // Simulate processing time
        }

        function processData() {
            const inputData = document.getElementById('inputData').value;
            const lines = inputData.split('\n');
            let data = [];
            let successCount = 0;
            let failCount = 0;
            let otherCount = 0;

            lines.forEach((line, index) => {
                const match = line.match(/(\d+), (\d{2}-\d{2}-\d{4}) (\d{2}:\d{2}) Tài khoản (\d+) nhắn tin cho bạn bè có user ID (\d+) (thành công|thất bại|.+)/);
                if (match) {
                    const [_, stt, date, time, account, uid, status] = match;
                    let statusCode = '';
                    if (status === 'thành công') {
                        statusCode = 1;
                        successCount++;
                    } else if (status === 'thất bại') {
                        statusCode = 0;
                        failCount++;
                    } else {
                        statusCode = 'Null';
                        otherCount++;
                    }
                    data.push({ stt: index + 1, date, time, uid, status: statusCode });
                }
            });

            const total = successCount + failCount + otherCount;
            const successRate = ((successCount / total) * 100).toFixed(2);
            const failRate = ((failCount / total) * 100).toFixed(2);
            const otherRate = ((otherCount / total) * 100).toFixed(2);

            document.getElementById('stats').innerHTML = `
                Total IDs: ${total}<br>
                Success: ${successCount} (${successRate}%)<br>
                Fail: ${failCount} (${failRate}%)<br>
                Other: ${otherCount} (${otherRate}%)
            `;

            const tbody = document.getElementById('resultTable').querySelector('tbody');
            tbody.innerHTML = '';
            data.forEach(row => {
                const tr = document.createElement('tr');
                tr.innerHTML = `<td>${row.stt}</td><td>${row.date}</td><td>${row.time}</td><td>${row.uid}</td><td>${row.status}</td>`;
                tbody.appendChild(tr);
            });

            document.getElementById('resultSection').style.display = 'block';

            const ctx = document.getElementById('chart').getContext('2d');
            new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: ['Success', 'Fail', 'Other'],
                    datasets: [{
                        data: [successRate, failRate, otherRate],
                        backgroundColor: ['#28a745', '#dc3545', '#ffc107']
                    }]
                }
            });
            document.getElementById('chartSection').style.display = 'block';
        }

        function downloadExcel() {
            const rows = [['STT', 'Date', 'Time', 'ID', 'Status']];
            const tableRows = document.querySelectorAll('#resultTable tbody tr');
            tableRows.forEach(row => {
                const cols = row.querySelectorAll('td');
                const rowData = [];
                cols.forEach(col => rowData.push(col.innerText));
                rows.push(rowData);
            });

            let csvContent = "data:text/csv;charset=utf-8," 
                + rows.map(e => e.join(",")).join("\n");

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "processed_data.csv");
            document.body.appendChild(link); // Required for FF

            link.click();
            document.body.removeChild(link);
        }
    </script>
</body>
</html>
