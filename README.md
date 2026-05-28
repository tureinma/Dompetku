<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dompetku - Catatan Keuangan</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        .container {
            width: 100%;
            max-width: 500px;
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        h2 { text-align: center; color: #333; }
        .balance-box {
            background: #eef2f7;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
            margin-bottom: 20px;
        }
        .stats {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
        }
        .stat-box {
            flex: 1;
            padding: 10px;
            text-align: center;
            border-radius: 8px;
            margin: 0 5px;
            color: white;
        }
        .income { background-color: #2ecc71; }
        .expense { background-color: #e74c3c; }
        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 20px;
        }
        input, select, button {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 6px;
            font-size: 16px;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover { background-color: #2980b9; }
        .history-list {
            list-style: none;
            padding: 0;
        }
        .history-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            border-bottom: 1px solid #eee;
            margin-bottom: 5px;
            border-radius: 4px;
        }
        .item-income { border-left: 5px solid #2ecc71; background: #f0fbf5; }
        .item-expense { border-left: 5px solid #e74c3c; background: #fdf2f1; }
        .delete-btn {
            background: none;
            border: none;
            color: #e74c3c;
            cursor: pointer;
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>💰 Dompetku</h2>
    
    <div class="balance-box">
        <small>Sisa Saldo Anda</small>
        <h1 id="total-balance">Rp 0</h1>
    </div>

    <div class="stats">
        <div class="stat-box income">
            <small>Pemasukan</small>
            <div id="total-income">Rp 0</div>
        </div>
        <div class="stat-box expense">
            <small>Pengeluaran</small>
            <div id="total-expense">Rp 0</div>
        </div>
    </div>

    <form id="finance-form">
        <input type="text" id="text-input" placeholder="Nama transaksi (ex: Beli Kopi)" required>
        <input type="number" id="amount-input" placeholder="Nominal (Rp)" required>
        <select id="type-input">
            <option value="income">Pemasukan</option>
            <option value="expense">Pengeluaran</option>
        </select>
        <button type="submit">Tambah Transaksi</button>
    </form>

    <h3>Riwayat Transaksi</h3>
    <ul id="history-list" class="history-list">
        </ul>
</div>

<script>
    // 1. Ambil semua elemen HTML yang kita butuhkan
    const financeForm = document.getElementById('finance-form');
    const textInput = document.getElementById('text-input');
    const amountInput = document.getElementById('amount-input');
    const typeInput = document.getElementById('type-input');
    const historyList = document.getElementById('history-list');
    const totalBalance = document.getElementById('total-balance');
    const totalIncome = document.getElementById('total-income');
    const totalExpense = document.getElementById('total-expense');

    // 2. Wadah tempat menyimpan semua data transaksi (berupa Array)
    let transactions = [];

    // 3. Fungsi untuk menghitung saldo, pemasukan, dan pengeluaran
    function updateValues() {
        let income = 0;
        let expense = 0;

        transactions.forEach(function(trx) {
            if (trx.type === 'income') {
                income += trx.amount;
            } else {
                expense += trx.amount;
            }
        });

        const balance = income - expense;

        // Tampilkan angka hasil hitungan ke layar HTML
        totalBalance.innerText = 'Rp ' + balance.toLocaleString('id-ID');
        totalIncome.innerText = 'Rp ' + income.toLocaleString('id-ID');
        totalExpense.innerText = 'Rp ' + expense.toLocaleString('id-ID');
    }

    // 4. Fungsi untuk menampilkan daftar riwayat di layar
    function renderList() {
        // Kosongkan list lama agar tidak menumpuk saat di-update
        historyList.innerHTML = '';

        transactions.forEach(function(trx, index) {
            const li = document.createElement('li');
            
            // Beri warna background berbeda berdasarkan jenis transaksi
            li.classList.add('history-item');
            if (trx.type === 'income') {
                li.classList.add('item-income');
            } else {
                li.classList.add('item-expense');
            }

            // Atur tanda + atau -
            let sign = trx.type === 'income' ? '+' : '-';

            // Isi dari baris riwayat
            li.innerHTML = '<span>' + trx.text + '</span>' +
                           '<span>' + sign + ' Rp ' + trx.amount.toLocaleString('id-ID') + ' ' +
                           '<button class="delete-btn" onclick="deleteTransaction(' + index + ')">❌</button></span>';

            historyList.appendChild(li);
        });
    }

    // 5. Fungsi untuk menambah transaksi baru saat tombol diklik
    financeForm.addEventListener('submit', function(event) {
        event.preventDefault(); // Mencegah halaman reload otomatis

        // Buat objek data transaksi baru
        const newTransaction = {
            text: textInput.value,
            amount: parseInt(amountInput.value),
            type: typeInput.value
        };

        // Masukkan ke dalam wadah array
        transactions.push(newTransaction);

        // Perbarui tampilan layar
        updateValues();
        renderList();

        // Kosongkan kembali form input
        textInput.value = '';
        amountInput.value = '';
    });

    // 6. Fungsi untuk menghapus transaksi
    function deleteTransaction(index) {
        // Hapus data berdasarkan urutan (index) nya
        transactions.splice(index, 1);
        
        // Perbarui ulang tampilan layar
        updateValues();
        renderList();
    }
</script>

</body>
</html>
