
 <html lang="en">
 <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>CSV Analysis</title>
     <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
     <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.4/pdfmake.min.js"></script>
     <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.4/vfs_fonts.js"></script>
     <style>
         body { font-family: Arial, sans-serif; text-align: center; }
         #chartContainer { display: flex; justify-content: space-around; margin-top: 20px; }
         canvas { max-width: 400px; }
         table { width: 80%; margin: 20px auto; border-collapse: collapse; }
         th, td { border: 1px solid black; padding: 8px; text-align: center; }
         th { background-color: #f2f2f2; }


      /* Style buttons */
.btn {
  background-color: DodgerBlue;
  border: none;
  color: white;
  padding: 12px 30px;
  cursor: pointer;
  font-size: 20px;
}

/* Darker background on mouse-over */
.btn:hover {
  background-color: RoyalBlue;
}

.download-button {
      font-size: 14px;
      padding: 8px 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      text-decoration: none;
    }
    .download-button:hover {
      background-color: #45a049;
    }
     </style>
 </head>
 <body>
     <h1>CSV Analysis Tool</h1>
     <input type="file" id="fileInput" accept=".csv">
     <button onclick="exportToPDF()">Export to PDF</button>
     <div id="chartContainer">
         <div>
             <h3>Subject Area Analysis</h3>
             <canvas id="subjectChart"></canvas>
         </div>
         <div>
             <h3>Question Type Analysis</h3>
             <canvas id="questionChart"></canvas>
         </div>
     </div>
     <h2>Analysis Table</h2>
     <table id="resultsTable"></table>

      <a href="https://github.com/mrcmcalpine/ExamAnalysis/raw/refs/heads/main/SampleFile.xlsx" download class="download-button">Download Template</a>

     
     <script>
         document.getElementById('fileInput').addEventListener('change', handleFile);
         
         function handleFile(event) {
             const file = event.target.files[0];
             if (!file) return;
             
             const reader = new FileReader();
             reader.onload = function(e) {
                 const lines = e.target.result.split('\n').map(line => line.split(','));
                 lines.shift(); // Ignore headers
                 processCSV(lines);
             };
             reader.readAsText(file);
         }
         
         function processCSV(data) {
             const subjects = {};
             const questionTypes = {};
             
             data.forEach(row => {
                 if (row.length < 4) return;
                 let [subject, questionType, marks, totalMarks] = row;
                 marks = parseInt(marks);
                 totalMarks = parseInt(totalMarks);
                 
                 if (!subjects[subject]) subjects[subject] = { achieved: 0, possible: 0 };
                 subjects[subject].achieved += marks;
                 subjects[subject].possible += totalMarks;
                 
                 if (!questionTypes[questionType]) questionTypes[questionType] = { achieved: 0, possible: 0 };
                 questionTypes[questionType].achieved += marks;
                 questionTypes[questionType].possible += totalMarks;
             });
             
             normalizeData(subjects);
             normalizeData(questionTypes);
             
             createRadarChart(subjects, 'subjectChart', 'Subject Area Analysis', 'rgba(54, 162, 235, 0.2)', 'rgba(54, 162, 235, 1)');
             createRadarChart(questionTypes, 'questionChart', 'Question Type Analysis', 'rgba(75, 192, 75, 0.2)', 'rgba(75, 192, 75, 1)');
             createTable(subjects, questionTypes);
         }
         
         function normalizeData(data) {
             Object.keys(data).forEach(key => {
                 if (data[key].possible > 0) {
                     data[key].percentage = (data[key].achieved / data[key].possible) * 100;
                 } else {
                     data[key].percentage = 0;
                 }
             });
         }
         
         function createRadarChart(data, canvasId, label, bgColor, borderColor) {
             const ctx = document.getElementById(canvasId).getContext('2d');
             new Chart(ctx, {
                 type: 'radar',
                 data: {
                     labels: Object.keys(data),
                     datasets: [{
                         label: label,
                         data: Object.values(data).map(d => d.percentage),
                         fill: true,
                         backgroundColor: bgColor,
                         borderColor: borderColor,
                         pointBackgroundColor: borderColor
                     }]
                 },
                 options: {
                     scales: {
                         r: {
                             suggestedMin: 0,
                             suggestedMax: 100,
                             pointLabels: { font: { weight: 'bold' } }
                         }
                     }
                 }
             });
         }
         
         function createTable(subjects, questionTypes) {
             let table = '<tr><th>Category</th><th>Achieved</th><th>Possible</th><th>Percentage</th></tr>';
             Object.keys(subjects).forEach(key => {
                 table += `<tr><td>${key}</td><td>${subjects[key].achieved}</td><td>${subjects[key].possible}</td><td>${subjects[key].percentage.toFixed(2)}%</td></tr>`;
             });
             Object.keys(questionTypes).forEach(key => {
                 table += `<tr><td>${key}</td><td>${questionTypes[key].achieved}</td><td>${questionTypes[key].possible}</td><td>${questionTypes[key].percentage.toFixed(2)}%</td></tr>`;
             });
             document.getElementById('resultsTable').innerHTML = table;
         }
     </script>
 </body>
 </html>
