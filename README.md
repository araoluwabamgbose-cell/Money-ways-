# Money-ways-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My Money Tracker</title>
<style>
* {
    box-sizing: border-box;
}
body {
    margin: 0;
    padding: 20px;
    font-family: Arial, sans-serif;
    background: #101827;
    color: white;
}
.container {
    max-width: 700px;
    margin: auto;
}
h1 {
    text-align: center;
    color: #5eead4;
}
.subtitle {
    text-align: center;
    color: #cbd5e1;
}
.card {
    background: #1e293b;
    padding: 20px;
    border-radius: 14px;
    margin: 18px 0;
}
.balance {
    text-align: center;
    font-size: 32px;
    color: #5eead4;
    font-weight: bold;
    overflow-wrap: anywhere;
}
.summary {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 12px;
}
.summary .card {
    margin: 0;
}
.income {
    color: #4ade80;
}
.expense {
    color: #fb7185;
}
input, select, button {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    border: none;
    border-radius: 8px;
    font-size: 15px;
}
input, select {
    background: #334155;
    color: white;
}
button {
    background: #14b8a6;
    color: #062a28;
    font-weight: bold;
    cursor: pointer;
}
button:hover {
    opacity: 0.85;
}
.delete {
    background: #fb7185;
    color: #3f0713;
    width: auto;
    padding: 8px 12px;
    margin: 0;
}
.transaction {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 12px 0;
    border-bottom: 1px solid #475569;
}
.transaction div {
    overflow-wrap: anywhere;
}
.small {
    color: #cbd5e1;
    font-size: 13px;
}
.progress {
    height: 12px;
    background: #334155;
    border-radius: 20px;
    overflow: hidden;
    margin-top: 12px;
}
.progress-bar {
    height: 100%;
    width: 0%;
    background: #5eead4;
}
.goal-info {
    display: flex;
    justify-content: space-between;
    gap: 10px;
    margin-top: 12px;
}
footer {
    text-align: center;
    color: #94a3b8;
    margin-top: 30px;
    font-size: 13px;
}
@media (max-width: 400px) {
    body {
        padding: 12px;
    }
    .transaction {
        align-items: flex-start;
    }
    .balance {
        font-size: 26px;
    }
}
</style>
</head>
<body>
<div class="container">
<h1>💰 My Money Tracker</h1>
<p class="subtitle">Manage your money. Build your future.</p>
<div class="card">
    <p>Total Balance</p>
    <div class="balance" id="balance">₦0.00</div>
</div>
<div class="summary">
    <div class="card">
        <p>Total Income</p>
        <h2 class="income" id="income">₦0.00</h2>
    </div>
    <div class="card">
        <p>Total Expenses</p>
        <h2 class="expense" id="expenses">₦0.00</h2>
    </div>
</div>
<div class="card">
    <h2>➕ Add Transaction</h2>
    <form id="transactionForm">
        <input
            type="text"
            id="description"
            placeholder="Description (e.g. Allowance)"
            maxlength="80"
            required
        >
        <input
            type="number"
            id="amount"
            placeholder="Amount in Naira"
            min="0.01"
            max="1000000000000"
            step="0.01"
            required
        >
        <select id="type">
            <option value="income">Income</option>
            <option value="expense">Expense</option>
        </select>
        <button type="submit">Add Transaction</button>
    </form>
</div>
<div class="card">
    <h2>🎯 My Savings Goal</h2>
    <input
        type="text"
        id="goalName"
        placeholder="What are you saving for?"
        maxlength="80"
    >
    <input
        type="number"
        id="goalAmount"
        placeholder="Target amount in Naira"
        min="0.01"
        max="1000000000000"
        step="0.01"
    >
    <button id="saveGoal" type="button">Save Goal</button>
    <div class="goal-info">
        <span id="goalLabel">No goal set</span>
        <span id="goalPercent">0%</span>
    </div>
    <div class="progress">
        <div class="progress-bar" id="progressBar"></div>
    </div>
    <p class="small" id="goalRemaining">Set a goal to start tracking.</p>
</div>
<div class="card">
    <h2>📋 Recent Transactions</h2>
    <div id="transactionList">No transactions yet.</div>
</div>
<footer>
    Your money. Your goals. Your future. 🚀
</footer>
</div>
<script>
const STORAGE_KEY = "myMoneyTracker_v1";
const defaultData = {
    transactions: [],
    goalName: "",
    goalAmount: 0
};
let data = loadData();
function loadData() {
    try {
        const saved = localStorage.getItem(STORAGE_KEY);
        if (!saved) return structuredClone(defaultData);
        const parsed = JSON.parse(saved);
        if (!Array.isArray(parsed.transactions)) {
            return structuredClone(defaultData);
        }
        return {
            transactions: parsed.transactions.filter(t =>
                t &&
                typeof t.id === "string" &&
                typeof t.description === "string" &&
                Number.isFinite(t.amount) &&
                t.amount > 0 &&
                ["income", "expense"].includes(t.type)
            ),
            goalName: typeof parsed.goalName === "string"
                ? parsed.goalName.slice(0, 80) : "",
            goalAmount: Number.isFinite(parsed.goalAmount) &&
                parsed.goalAmount > 0 ? parsed.goalAmount : 0
        };
    } catch {
        return structuredClone(defaultData);
    }
}
function saveData() {
    try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
        return true;
    } catch {
        alert("Could not save data. Your browser storage may be full or unavailable.");
        return false;
    }
}
function money(amount) {
    return new Intl.NumberFormat("en-NG", {
        style: "currency",
        currency: "NGN"
    }).format(amount);
}
function totals() {
    let income = 0;
    let expenses = 0;
    data.transactions.forEach(t => {
        if (t.type === "income") income += t.amount;
        else expenses += t.amount;
    });
    return {
        income,
        expenses,
        balance: income - expenses
    };
}
document.getElementById("transactionForm")
.addEventListener("submit", function(event) {
    event.preventDefault();
    const description = document.getElementById("description")
        .value.trim();
    const amount = Number(document.getElementById("amount").value);
    const type = document.getElementById("type").value;
    if (!description || !Number.isFinite(amount) ||
        amount <= 0 || amount > 1e12) {
        alert("Enter a valid description and amount.");
        return;
    }
    data.transactions.unshift({
        id: crypto.randomUUID(),
        description,
        amount,
        type,
        date: new Date().toLocaleDateString()
    });
    if (!saveData()) {
        data.transactions.shift();
        return;
    }
    this.reset();
    render();
});
document.getElementById("saveGoal")
.addEventListener("click", function() {
    const name = document.getElementById("goalName")
        .value.trim();
    const amount = Number(document.getElementById("goalAmount").value);
    if (!name || !Number.isFinite(amount) ||
        amount <= 0 || amount > 1e12) {
        alert("Enter a goal name and a valid target amount.");
        return;
    }
    const previous = {
        goalName: data.goalName,
        goalAmount: data.goalAmount
    };
    data.goalName = name;
    data.goalAmount = amount;
    if (!saveData()) {
        Object.assign(data, previous);
        return;
    }
    render();
});
function deleteTransaction(id) {
    const previous = data.transactions;
    data.transactions = previous.filter(t => t.id !== id);
    if (!saveData()) {
        data.transactions = previous;
        return;
    }
    render();
}
function render() {
    const t = totals();
    document.getElementById("balance").textContent = money(t.balance);
    document.getElementById("income").textContent = money(t.income);
    document.getElementById("expenses").textContent = money(t.expenses);
    const list = document.getElementById("transactionList");
    list.replaceChildren();
    if (data.transactions.length === 0) {
        list.textContent = "No transactions yet.";
    } else {
        data.transactions.forEach(item => {
            const row = document.createElement("div");
            row.className = "transaction";
            const details = document.createElement("div");
            const title = document.createElement("strong");
            title.textContent = item.description;
            const date = document.createElement("div");
            date.className = "small";
            date.textContent = item.date || "";
            const amount = document.createElement("div");
            amount.className = item.type;
            amount.textContent =
                (item.type === "income" ? "+" : "-") +
                money(item.amount);
            details.append(title, date, amount);
            const remove = document.createElement("button");
            remove.className = "delete";
            remove.type = "button";
            remove.textContent = "Delete";
            remove.setAttribute(
                "aria-label",
                "Delete transaction: " + item.description
            );
            remove.addEventListener("click", () => {
                deleteTransaction(item.id);
            });
            row.append(details, remove);
            list.appendChild(row);
        });
    }
    document.getElementById("goalName").value = data.goalName;
    document.getElementById("goalAmount").value =
        data.goalAmount || "";
    document.getElementById("goalLabel").textContent =
        data.goalName || "No goal set";
    const saved = Math.max(0, t.balance);
    const target = data.goalAmount;
    const percent = target > 0
        ? Math.min(100, Math.floor(saved / target * 100))
        : 0;
    document.getElementById("goalPercent").textContent =
        percent + "%";
    document.getElementById("progressBar").style.width =
        percent + "%";
    document.getElementById("goalRemaining").textContent =
        target > 0
            ? (saved >= target
                ? "🎉 Goal reached! Keep going!"
                : money(target - saved) + " left to reach your goal.")
            : "Set a goal to start tracking.";
}
render();
</script>
</body>
</html>
