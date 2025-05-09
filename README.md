
 <html lang="en">
 <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>CSV Analysis</title>
     <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
     <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.4/pdfmake.min.js"></script>
     <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.4/vfs_fonts.js"></script>


  <!-- Add these to the HTML head if not already included -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<script>
async function capturePageToPDF() {
  const { jsPDF } = window.jspdf;
  const fileName = prompt("Enter a name for the PDF file:");
  if (!fileName) return;

  // Temporarily add the header
  const header = document.createElement('div');
  header.textContent = fileName;
  header.style.fontSize = '24px';
  header.style.fontWeight = 'bold';
  header.style.margin = '20px';
  header.style.textAlign = 'center';
  document.body.insertBefore(header, document.body.firstChild);

  // Wait for header to render
  await new Promise(resolve => setTimeout(resolve, 100));

  // Capture screenshot of the body
  const canvas = await html2canvas(document.body, {
    useCORS: true,
    scale: 2
  });

  const imgData = canvas.toDataURL('image/png');

  const pdf = new jsPDF({
    orientation: 'portrait',
    unit: 'mm',
    format: 'a4'
  });

  const pageWidth = pdf.internal.pageSize.getWidth();
  const pageHeight = pdf.internal.pageSize.getHeight();

  const imgProps = pdf.getImageProperties(imgData);
  const pdfWidth = pageWidth;
  const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;

  pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);

  pdf.save(fileName + ".pdf");

  // Clean up header after save
  header.remove();
}
</script>
     <style>
         body { font-family: Arial, sans-serif; text-align: center; }
         #chartContainer {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
  gap: 40px;
}
canvas {
  width: 100% !important;
  max-width: 500px;
  height: auto !important;
}
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
     <h1>CSV Analysis Tool - April 2025</h1>
     <input type="file" id="fileInput" accept=".csv">
     <button onclick="capturePageToPDF()">Export to PDF</button>
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
    

     <h2>Subject Analysis</h2>
<table id="subjectTable"></table>

<h2>Question Type Analysis</h2>
<table id="questionTypeTable"></table>

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
            pointLabels: {
                font: {
                    size: 16,      // 🔍 Make labels bigger
                    weight: 'bold'
                },
                color: '#000'    // Optional: make sure they're readable
            },
            ticks: {
                backdropColor: 'transparent', // Removes the white boxes behind numbers
                font: {
                    size: 12
                }
            }
        }
    },
    plugins: {
        legend: {
            labels: {
                font: {
                    size: 14
                }
            }
        }
    }
}
             });
         }
         
         function createTable(subjects, questionTypes) {
    let subjectTable = '<tr><th>Subject</th><th>Achieved</th><th>Possible</th><th>Percentage</th></tr>';
    Object.keys(subjects).forEach(key => {
        subjectTable += `<tr><td>${key}</td><td>${subjects[key].achieved}</td><td>${subjects[key].possible}</td><td>${subjects[key].percentage.toFixed(2)}%</td></tr>`;
    });
    document.getElementById('subjectTable').innerHTML = subjectTable;

    let questionTable = '<tr><th>Question Type</th><th>Achieved</th><th>Possible</th><th>Percentage</th></tr>';
    Object.keys(questionTypes).forEach(key => {
        questionTable += `<tr><td>${key}</td><td>${questionTypes[key].achieved}</td><td>${questionTypes[key].possible}</td><td>${questionTypes[key].percentage.toFixed(2)}%</td></tr>`;
    });
    document.getElementById('questionTypeTable').innerHTML = questionTable;
}
     </script>
 </body>
 </html>
