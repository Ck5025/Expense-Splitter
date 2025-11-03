# Expense-Splitter
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Professional Expense Splitter</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto&display=swap" rel="stylesheet">
<style>
body {
    font-family: 'Roboto', sans-serif;
    background: linear-gradient(to right, #eef2f3, #8e9eab);
    margin: 0; 
    padding: 0;
}
header {
    background: #5c6bc0;
    color: white;
    text-align: center;
    padding: 20px 0;
    font-size: 2em;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
section {
    background: white;
    border-radius: 12px;
    max-width: 650px;
    margin: 20px auto;
    padding: 20px;
    box-shadow: 0 6px 10px rgba(0,0,0,0.15);
}
input, select, button {
    width: 100%;
    padding: 10px;
    margin: 5px 0 15px 0;
    border-radius: 8px;
    border: 1px solid #ccc;
    font-size: 1em;
    box-sizing: border-box;
}
button {
    background: #3949ab;
    color: white;
    font-weight: bold;
    cursor: pointer;
    transition: background 0.3s ease;
}
button:hover {
    background: #283593;
}
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
}
th, td {
    border: 1px solid #ddd;
    padding: 10px;
    text-align: center;
}
th {
    background: #5c6bc0;
    color: white;
}
ul {
    list-style: none;
    padding-left: 0;
}
li {
    background: #f1f3f6;
    padding: 8px 10px;
    margin: 5px 0;
    border-radius: 6px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}
li span.delete {
    cursor: pointer;
    color: red;
    font-weight: bold;
}
.export-buttons {
    display: flex;
    justify-content: space-between;
    gap: 10px;
}
.export-buttons button {
    width: 48%;
}
footer {
    background: #3949ab;
    color: white;
    text-align: center;
    padding: 20px 10px;
    font-size: 0.95em;
    border-top: 3px solid #283593;
}
footer a {
    color: #c5cae9;
    text-decoration: none;
}
footer a:hover {
    text-decoration: underline;
}
</style>
</head>
<body>

<header>Expense Splitter</header>

<section>
<h2>Add Participants</h2>
<input type="text" id="participantName" placeholder="Enter participant name">
<button onclick="addParticipant()">Add Participant</button>
<ul id="participantsList"></ul>
</section>

<section>
<h2>Add Expense</h2>
<input type="text" id="expenseName" placeholder="Expense name">
<input type="number" id="expenseAmount" placeholder="Amount">
<select id="expenseCategory">
    <option value="">Select category</option>
    <option value="Food">Food</option>
    <option value="Accommodation">Accommodation</option>
    <option value="Travel">Travel</option>
    <option value="Other">Other</option>
</select>
<select id="paidBy"></select>
<label>Select participants sharing this expense:</label>
<select id="sharedWith" multiple size="5"></select>
<button onclick="addExpense()">Add Expense</button>
<ul id="expensesList"></ul>
</section>

<section>
<h2>Net Balances</h2>
<table>
    <thead>
        <tr>
            <th>Participant</th>
            <th>Net Balance</th>
        </tr>
    </thead>
    <tbody id="summaryTable"></tbody>
</table>
</section>

<section>
<h2>Final Settlements</h2>
<ul id="settlementList"></ul>
</section>

<section class="export-buttons">
<button onclick="exportCSV()">Export CSV</button>
<button onclick="exportPDF()">Export PDF</button>
</section>

<footer>
    <p><strong>Terms of Use:</strong> This Expense Splitter tool is designed for personal and educational use only. Users are responsible for verifying the accuracy of all data entered. Unauthorized duplication or distribution of this code or design is prohibited.</p>
    <p>© 2025 <strong>K Chandra Kamali</strong>. All rights reserved.</p>
</footer>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
let participants = [];
let expenses = [];

// Update participant list in UI
function updateParticipantsUI() {
    const list = document.getElementById("participantsList");
    const paidBySelect = document.getElementById("paidBy");
    const sharedWithSelect = document.getElementById("sharedWith");

    list.innerHTML = "";
    paidBySelect.innerHTML = "";
    sharedWithSelect.innerHTML = "";

    participants.forEach(name => {
        // Participant list with delete icon
        let li = document.createElement("li");
        li.textContent = name;
        let del = document.createElement("span");
        del.textContent = "🗑";
        del.classList.add("delete");
        del.onclick = () => removeParticipant(name);
        li.appendChild(del);
        list.appendChild(li);

        // Options for paidBy and sharedWith
        let option1 = document.createElement("option");
        option1.value = name; option1.textContent = name;
        paidBySelect.appendChild(option1);

        let option2 = document.createElement("option");
        option2.value = name; option2.textContent = name;
        sharedWithSelect.appendChild(option2);
    });
}

// Add participant
function addParticipant() {
    const nameInput = document.getElementById("participantName");
    const name = nameInput.value.trim();
    if(name && !participants.includes(name)) {
        participants.push(name);
        updateParticipantsUI();
        nameInput.value = "";
        updateSummary();
        calculateSettlements();
    } else {
        alert("Enter a unique participant name.");
    }
}

// Remove participant
function removeParticipant(name) {
    if(confirm(`Remove participant ${name}? This will remove related expenses too.`)){
        participants = participants.filter(p => p !== name);
        expenses = expenses.filter(e => e.paidBy !== name && !e.sharedWith.includes(name));
        updateParticipantsUI();
        updateExpensesUI();
    }
}

// Update expenses UI
function updateExpensesUI() {
    const list = document.getElementById("expensesList");
    list.innerHTML = "";
    expenses.forEach((exp,index) => {
        let li = document.createElement("li");
        li.textContent = `${exp.name} - $${exp.amount.toFixed(2)} [${exp.category}] (Paid by: ${exp.paidBy}, Shared with: ${exp.sharedWith.join(", ")})`;
        let del = document.createElement("span");
        del.textContent = "🗑";
        del.classList.add("delete");
        del.onclick = () => removeExpense(index);
        li.appendChild(del);
        list.appendChild(li);
    });
    updateSummary();
    calculateSettlements();
}

// Remove expense
function removeExpense(index) {
    expenses.splice(index,1);
    updateExpensesUI();
}

// Add expense
function addExpense() {
    const name = document.getElementById("expenseName").value.trim();
    const amount = parseFloat(document.getElementById("expenseAmount").value);
    const category = document.getElementById("expenseCategory").value;
    const paidBy = document.getElementById("paidBy").value;
    const sharedWithSelect = document.getElementById("sharedWith");
    const sharedWith = Array.from(sharedWithSelect.selectedOptions).map(o => o.value);

    if(!name || isNaN(amount) || amount <= 0 || !paidBy || sharedWith.length === 0 || !category) {
        alert("Please fill all fields correctly.");
        return;
    }

    expenses.push({ name, amount, category, paidBy, sharedWith });
    document.getElementById("expenseName").value = "";
    document.getElementById("expenseAmount").value = "";
    document.getElementById("expenseCategory").value = "";
    updateExpensesUI();
}

// Update net balance table
function updateSummary() {
    let balances = {};
    participants.forEach(p => balances[p] = 0);

    expenses.forEach(exp => {
        const splitAmount = exp.amount / exp.sharedWith.length;
        exp.sharedWith.forEach(person => {
            if(person !== exp.paidBy) balances[person] -= splitAmount;
            else balances[person] += exp.amount - splitAmount;
        });
    });

    const tbody = document.getElementById("summaryTable");
    tbody.innerHTML = "";
    participants.forEach(p => {
        let tr = document.createElement("tr");
        let tdName = document.createElement("td"); tdName.textContent = p;
        let tdBalance = document.createElement("td");
        tdBalance.textContent = `$${balances[p].toFixed(2)}`;
        tdBalance.style.color = balances[p]>=0?"green":"red";
        tr.appendChild(tdName); tr.appendChild(tdBalance);
        tbody.appendChild(tr);
    });
}

// Calculate settlements
function calculateSettlements() {
    let balances = {};
    participants.forEach(p => balances[p] = 0);
    expenses.forEach(exp => {
        const splitAmount = exp.amount / exp.sharedWith.length;
        exp.sharedWith.forEach(person => {
            if(person !== exp.paidBy) balances[person] -= splitAmount;
            else balances[person] += exp.amount - splitAmount;
        });
    });

    let creditors=[], debtors=[];
    for(let person in balances){
        let amt = parseFloat(balances[person].toFixed(2));
        if(amt>0) creditors.push({name:person,amount:amt});
        else if(amt<0) debtors.push({name:person,amount:-amt});
    }

    let settlements = [];
    let i=0,j=0;
    while(i<debtors.length && j<creditors.length){
        let debtor = debtors[i]; let creditor = creditors[j];
        let payAmount = Math.min(debtor.amount, creditor.amount);
        settlements.push(`${debtor.name} pays ${creditor.name} $${payAmount.toFixed(2)}`);
        debtor.amount -= payAmount; creditor.amount -= payAmount;
        if(debtor.amount===0)i++;
        if(creditor.amount===0)j++;
    }

    const ul = document.getElementById("settlementList");
    ul.innerHTML="";
    if(settlements.length===0) ul.innerHTML="<li>All settled! No one owes anything.</li>";
    else settlements.forEach(s=>{
        let li = document.createElement("li");
        li.textContent = s;
        ul.appendChild(li);
    });
}

// Export CSV
function exportCSV() {
    let csv="Participants\nName\n";
    participants.forEach(p=>csv+=p+"\n");
    csv+="\nExpenses\nName,Amount,Category,PaidBy,SharedWith\n";
    expenses.forEach(e=>{
        csv+=`${e.name},${e.amount},${e.category},${e.paidBy},"${e.sharedWith.join(';')}"\n`;
    });
    csv+="\nNet Balances\nParticipant,Balance\n";
    let balances={}; participants.forEach(p=>balances[p]=0);
    expenses.forEach(exp=>{
        const splitAmount=exp.amount/exp.sharedWith.length;
        exp.sharedWith.forEach(person=>{
            if(person!==exp.paidBy) balances[person]-=splitAmount;
            else balances[person]+=exp.amount-splitAmount;
        });
    });
    for(let p in balances) csv+=`${p},${balances[p].toFixed(2)}\n`;
    csv+="\nSettlements\n";
    const ul = document.getElementById("settlementList");
    Array.from(ul.children).forEach(li=>{
        csv+=li.textContent+"\n";
    });
    const blob=new Blob([csv],{type:"text/csv"});
    const url=URL.createObjectURL(blob);
    const a=document.createElement("a");
    a.href=url; a.download="expense_report.csv"; a.click();
    URL.revokeObjectURL(url);
}

// Export PDF
function exportPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    doc.setFontSize(16);
    doc.text("Expense Report", 105, 15, null, null, "center");
    doc.setFontSize(12);
    let y=25;
    doc.text("Participants:", 10, y); y+=7;
    participants.forEach(p=>{doc.text("- "+p, 12, y); y+=6;});
    y+=4;
    doc.text("Expenses:", 10, y); y+=7;
    expenses.forEach(e=>{
        doc.text(`- ${e.name}, $${e.amount.toFixed(2)}, ${e.category}, PaidBy: ${e.paidBy}, SharedWith: ${e.sharedWith.join(", ")}`, 12, y);
        y+=6;
        if(y>270){ doc.addPage(); y=20; }
    });
    y+=4;
    doc.text("Net Balances:",10,y); y+=7;
    let balances={}; participants.forEach(p=>balances[p]=0);
    expenses.forEach(exp=>{
        const splitAmount=exp.amount/exp.sharedWith.length;
        exp.sharedWith.forEach(person=>{
            if(person!==exp.paidBy) balances[person]-=splitAmount;
            else balances[person]+=exp.amount-splitAmount;
        });
    });
    for(let p in balances){ doc.text(`- ${p}: $${balances[p].toFixed(2)}`,12,y); y+=6; if(y>270){ doc.addPage(); y=20; } }
    y+=4;
    doc.text("Settlements:",10,y); y+=7;
    const ul=document.getElementById("settlementList");
    Array.from(ul.children).forEach(li=>{
        doc.text("- "+li.textContent,12,y); y+=6; if(y>270){ doc.addPage(); y=20; }
    });
    doc.save("expense_report.pdf");
}
</script>

</body>
</html>
