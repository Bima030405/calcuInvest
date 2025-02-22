# calcuInvest
Membantu Menghitung Investasi Anda
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Kalkulator Keuangan</title>
  <style>
    body {
      background: #f7f7f7;
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .calculator-container {
      background: #fff;
      max-width: 600px;
      width: 100%;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
    h1, h2 {
      text-align: center;
    }
    label {
      display: block;
      margin: 10px 0 5px;
    }
    input {
      width: 100%;
      padding: 10px;
      margin-bottom: 10px;
      box-sizing: border-box;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    button {
      width: 100%;
      padding: 10px;
      background: #007BFF;
      border: none;
      border-radius: 4px;
      color: white;
      font-size: 16px;
      cursor: pointer;
      margin-top: 10px;
    }
    button:hover {
      background: #0056b3;
    }
    .result {
      background: #e9ecef;
      padding: 15px;
      margin-top: 20px;
      border-radius: 4px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    table, th, td {
      border: 1px solid #ccc;
    }
    th, td {
      padding: 8px;
      text-align: center;
    }
    th {
      background: #f2f2f2;
    }
  </style>
</head>
<body>
  <div class="calculator-container">
    <h1>Kalkulator Keuangan</h1>
    <label for="principal">Investasi Awal (Rp):</label>
    <input type="number" id="principal" placeholder="Masukkan investasi awal">

    <label for="contribution">Kontribusi Bulanan (Rp):</label>
    <input type="number" id="contribution" placeholder="Masukkan kontribusi bulanan">

    <label for="interest">Suku Bunga Tahunan (%):</label>
    <input type="number" id="interest" placeholder="Masukkan suku bunga tahunan" step="0.01">

    <label for="years">Periode Investasi (tahun):</label>
    <input type="number" id="years" placeholder="Masukkan periode investasi">

    <button onclick="calculateFinance()">Hitung Investasi</button>

    <div class="result" id="result"></div>
    <button id="toggleScheduleBtn" style="display: none;" onclick="toggleSchedule()">Tampilkan Jadwal Pertumbuhan Investasi</button>
  </div>

  <div class="calculator-container" id="scheduleContainer" style="display: none;">
    <h2>Jadwal Pertumbuhan Investasi</h2>
    <table id="scheduleTable">
      <thead>
        <tr>
          <th>Tahun</th>
          <th>Saldo Awal Tahun (Rp)</th>
          <th>Kontribusi Tahun (Rp)</th>
          <th>Bunga Tahun (Rp)</th>
          <th>Saldo Akhir Tahun (Rp)</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <script>
    function calculateFinance() {
      // Ambil nilai input
      var principal = parseFloat(document.getElementById("principal").value);
      var contribution = parseFloat(document.getElementById("contribution").value);
      var annualInterest = parseFloat(document.getElementById("interest").value);
      var years = parseFloat(document.getElementById("years").value);
      
      if (isNaN(principal) || isNaN(contribution) || isNaN(annualInterest) || isNaN(years) ||
          principal < 0 || contribution < 0 || annualInterest < 0 || years <= 0) {
        document.getElementById("result").innerHTML = "Silakan masukkan nilai yang valid.";
        document.getElementById("toggleScheduleBtn").style.display = "none";
        document.getElementById("scheduleContainer").style.display = "none";
        return;
      }
      
      var rDecimal = annualInterest / 100;
      var n = 12; // Perhitungan dilakukan tiap bulan
      var totalMonths = years * n;
      var monthlyRate = rDecimal / n;
      
      // Hitung nilai akhir investasi dengan kontribusi bulanan dan bunga majemuk:
      // Jika bunga > 0: FV = P*(1+monthlyRate)^(totalMonths) + contribution * [((1+monthlyRate)^(totalMonths)-1)/monthlyRate]
      // Jika bunga = 0: FV = P + contribution * totalMonths
      var futureValue;
      if (monthlyRate === 0) {
        futureValue = principal + contribution * totalMonths;
      } else {
        futureValue = principal * Math.pow(1 + monthlyRate, totalMonths) +
                      contribution * ((Math.pow(1 + monthlyRate, totalMonths) - 1) / monthlyRate);
      }
      futureValue = parseFloat(futureValue.toFixed(2));
      
      // Total kontribusi (investasi awal + kontribusi bulanan)
      var totalContributions = principal + contribution * totalMonths;
      totalContributions = parseFloat(totalContributions.toFixed(2));
      
      // Total bunga yang diperoleh
      var totalInterest = futureValue - totalContributions;
      totalInterest = parseFloat(totalInterest.toFixed(2));
      
      document.getElementById("result").innerHTML =
        "<strong>Nilai Akhir Investasi:</strong> Rp " + futureValue.toLocaleString("id-ID") + "<br>" +
        "<strong>Total Kontribusi:</strong> Rp " + totalContributions.toLocaleString("id-ID") + "<br>" +
        "<strong>Total Bunga:</strong> Rp " + totalInterest.toLocaleString("id-ID");
      
      // Hasilkan jadwal pertumbuhan investasi secara simulasi per tahun
      generateSchedule(principal, contribution, monthlyRate, totalMonths, years);
      document.getElementById("toggleScheduleBtn").style.display = "block";
    }
    
    // Fungsi untuk mensimulasikan pertumbuhan investasi per tahun
    function generateSchedule(principal, contribution, monthlyRate, totalMonths, years) {
      var tableBody = document.querySelector("#scheduleTable tbody");
      tableBody.innerHTML = "";
      
      var balance = principal;
      
      // Simulasikan perhitungan secara bulanan, dan catat hasil tiap akhir tahun
      for (var y = 1; y <= years; y++) {
        var startBalance = balance;
        var annualContribution = contribution * 12;
        var interestYear = 0;
        
        // Proses selama 12 bulan
        for (var m = 1; m <= 12; m++) {
          var interestMonth = balance * monthlyRate;
          interestYear += interestMonth;
          balance += interestMonth + contribution;
        }
        var endBalance = balance;
        
        var row = document.createElement("tr");
        row.innerHTML =
          "<td>" + y + "</td>" +
          "<td>Rp " + startBalance.toLocaleString("id-ID", {maximumFractionDigits:2}) + "</td>" +
          "<td>Rp " + annualContribution.toLocaleString("id-ID", {maximumFractionDigits:2}) + "</td>" +
          "<td>Rp " + interestYear.toLocaleString("id-ID", {maximumFractionDigits:2}) + "</td>" +
          "<td>Rp " + endBalance.toLocaleString("id-ID", {maximumFractionDigits:2}) + "</td>";
        tableBody.appendChild(row);
      }
      
      document.getElementById("scheduleContainer").style.display = "block";
    }
    
    // Fungsi untuk menampilkan atau menyembunyikan jadwal
    function toggleSchedule() {
      var scheduleContainer = document.getElementById("scheduleContainer");
      var btn = document.getElementById("toggleScheduleBtn");
      if (scheduleContainer.style.display === "none" || scheduleContainer.style.display === "") {
        scheduleContainer.style.display = "block";
        btn.innerText = "Sembunyikan Jadwal Pertumbuhan Investasi";
      } else {
        scheduleContainer.style.display = "none";
        btn.innerText = "Tampilkan Jadwal Pertumbuhan Investasi";
      }
    }
  </script>
</body>
</html>
