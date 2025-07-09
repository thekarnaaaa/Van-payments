# Van-payments
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Van Fee Payment Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Add Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-storage.js"></script>
    <style>
        @media print {
            .no-print {
                display: none !important;
            }
        }
        .payment-card:nth-child(odd) {
            background-color: #f8fafc;
        }
        .payment-card:nth-child(even) {
            background-color: #ffffff;
        }
    </style>
</head>
<body class="bg-gray-100">
    <div class="container mx-auto px-4 py-8 max-w-4xl">
        <div class="bg-white rounded-lg shadow-md overflow-hidden mb-8">
            <div class="bg-blue-600 px-6 py-4">
                <h1 class="text-2xl font-bold text-white">Van Fee Payment System</h1>
            </div>
            
            <!-- Payment Entry Form -->
            <div class="p-6 border-b">
                <h2 class="text-xl font-semibold mb-4">New Payment Entry</h2>
                <form id="paymentForm" class="space-y-4">
                    <div>
                        <label class="block text-gray-700 font-medium mb-2" for="studentName">Student Name</label>
                        <input type="text" id="studentName" required class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    
                    <div>
                        <label class="block text-gray-700 font-medium mb-2" for="mobileNumber">Mobile Number</label>
                        <input type="tel" id="mobileNumber" required class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    
                    <div>
                        <label class="block text-gray-700 font-medium mb-2" for="paymentMonth">Payment Month</label>
                        <select id="paymentMonth" required class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                            <option value="" disabled selected>Select Month</option>
                            <option value="January">January</option>
                            <option value="February">February</option>
                            <option value="March">March</option>
                            <option value="April">April</option>
                            <option value="May">May</option>
                            <option value="June">June</option>
                            <option value="July">July</option>
                            <option value="August">August</option>
                            <option value="September">September</option>
                            <option value="October">October</option>
                            <option value="November">November</option>
                            <option value="December">December</option>
                        </select>
                    </div>
                    
                    <div>
                        <label class="block text-gray-700 font-medium mb-2" for="paymentScreenshot">Payment Screenshot</label>
                        <input type="file" id="paymentScreenshot" accept="image/*" required class="w-full px-4 py-2 border rounded-lg">
                    </div>
                    
                    <div class="flex justify-end">
                        <button type="submit" class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition">Submit Payment</button>
                    </div>
                </form>
            </div>
            
            <!-- Admin Access (hidden by default) -->
            <div class="p-6 hidden" id="adminSection">
                <h2 class="text-xl font-semibold mb-4">Admin Approval Panel</h2>
                <div class="flex justify-between items-center mb-6">
                    <input type="text" id="searchInput" placeholder="Search by name or mobile" class="px-4 py-2 border rounded-lg w-2/3">
                    <select id="filterStatus" class="px-4 py-2 border rounded-lg">
                        <option value="all">All</option>
                        <option value="pending">Pending</option>
                        <option value="approved">Approved</option>
                    </select>
                </div>
                
                <div id="paymentsList" class="divide-y divide-gray-200">
                    <!-- Payments will be listed here -->
                </div>
            </div>
            
            <!-- Mode Toggle -->
            <div class="bg-gray-50 px-6 py-3 flex justify-end no-print">
                <button id="toggleAdminMode" class="text-blue-600 hover:text-blue-800 font-medium">Admin Mode</button>
            </div>
        </div>

        <!-- Admin Login Modal -->
        <div id="loginModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden no-print">
            <div class="bg-white rounded-lg w-full max-w-md overflow-hidden">
                <div class="bg-blue-600 px-6 py-4">
                    <h2 class="text-xl font-bold text-white">Admin Login</h2>
                </div>
                <div class="p-6">
                    <form id="loginForm" class="space-y-4">
                        <div>
                            <label class="block text-gray-700 font-medium mb-2" for="username">Username</label>
                            <input type="text" id="username" required class="w-full px-4 py-2 border rounded-lg">
                        </div>
                        <div>
                            <label class="block text-gray-700 font-medium mb-2" for="password">Password</label>
                            <input type="password" id="password" required class="w-full px-4 py-2 border rounded-lg">
                        </div>
                        <div class="flex justify-end">
                            <button type="submit" class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition">Login</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
        
        <!-- Print Receipt Modal (hidden by default) -->
        <div id="receiptModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden no-print">
            <div class="bg-white rounded-lg w-full max-w-md overflow-hidden">
                <div class="bg-blue-600 px-6 py-4">
                    <h2 class="text-xl font-bold text-white">Payment Receipt</h2>
                </div>
                <div class="p-6" id="receiptContent">
                    <!-- Receipt content will be inserted here -->
                </div>
                <div class="bg-gray-50 px-6 py-3 flex justify-end">
                    <button id="printReceipt" class="bg-blue-600 text-white px-4 py-2 rounded-lg mr-2">Print</button>
                    <button id="closeReceipt" class="bg-gray-200 text-gray-700 px-4 py-2 rounded-lg">Close</button>
                </div>
            </div>
        </div>
    </div>

   <script>
    // Firebase configuration
    const firebaseConfig = {
      apiKey: "AIzaSyCUZw65FBOLSSDX1xKg_3so750VWnjXFUs",
      authDomain: "vanpayment.firebaseapp.com",
      databaseURL: "https://vanpayment-default-rtdb.asia-southeast1.firebasedatabase.app",
      projectId: "vanpayment",
      storageBucket: "vanpayment.appspot.com",
      messagingSenderId: "158767920927",
      appId: "1:158767920927:web:70aa80f28df7ce7dfc622a",
      measurementId: "G-ZMDXBK65VJ"
    };

    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();
    const storage = firebase.storage();
        
        // Payment data storage
        let payments = [];
        
        // DOM Elements
        const paymentForm = document.getElementById('paymentForm');
        const adminSection = document.getElementById('adminSection');
        const toggleAdminMode = document.getElementById('toggleAdminMode');
        const paymentsList = document.getElementById('paymentsList');
        const searchInput = document.getElementById('searchInput');
        const filterStatus = document.getElementById('filterStatus');
        const receiptModal = document.getElementById('receiptModal');
        const receiptContent = document.getElementById('receiptContent');
        const printReceipt = document.getElementById('printReceipt');
        const closeReceipt = document.getElementById('closeReceipt');
        
        // Admin mode flag
        let isAdminMode = false;
        const ADMIN_CREDENTIALS = {
            username: 'harsh',
            password: '12122006'
        };

        // Toggle Admin Mode
        toggleAdminMode.addEventListener('click', () => {
            if (!isAdminMode) {
                // Show login modal when trying to enter admin mode
                document.getElementById('loginModal').classList.remove('hidden');
            } else {
                // Exit admin mode
                isAdminMode = false;
                toggleAdminMode.textContent = 'Admin Mode';
                adminSection.classList.add('hidden');
            }
        });

        // Handle login form submission
        document.getElementById('loginForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;

            if (username === ADMIN_CREDENTIALS.username && password === ADMIN_CREDENTIALS.password) {
                isAdminMode = true;
                toggleAdminMode.textContent = 'User  Mode';
                adminSection.classList.remove('hidden');
                renderPaymentsList();
                document.getElementById('loginModal').classList.add('hidden');
                document.getElementById('loginForm').reset();
            } else {
                alert('Invalid username or password');
            }
        });
        
        // Submit Payment Form
        paymentForm.addEventListener('submit', (e) => {
            e.preventDefault();
            
            const studentName = document.getElementById('studentName').value;
            const mobileNumber = document.getElementById('mobileNumber').value;
            const paymentMonth = document.getElementById('paymentMonth').value;
            const paymentScreenshot = document.getElementById('paymentScreenshot').files[0];
            
            if (!paymentScreenshot) {
                alert('Please upload payment screenshot');
                return;
            }
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const newPayment = {
                    id: Date.now(),
                    studentName,
                    mobileNumber,
                    paymentMonth,
                    paymentScreenshot: e.target.result,
                    status: 'pending',
                    date: new Date().toLocaleString()
                };
                
                payments.unshift(newPayment);
                savePayments();
                paymentForm.reset();
                
                if (!isAdminMode) {
                    generateReceipt(newPayment);
                    receiptModal.classList.remove('hidden');
                } else {
                    renderPaymentsList();
                }
            };
            reader.readAsDataURL(paymentScreenshot);
        });
        
        // Save to Firebase
        function savePayments() {
            database.ref('vanPayments').set(payments)
                .then(() => console.log('Data saved to Firebase'))
                .catch(error => console.error('Error saving data:', error));
        }

        // Load from Firebase
        function loadPayments() {
            database.ref('vanPayments').on('value', (snapshot) => {
                payments = snapshot.val() || [];
                renderPaymentsList();
            });
        }
        
        // Render payments list for admin
        function renderPaymentsList(filter = '') {
            const statusFilter = filterStatus.value;
            const searchTerm = filter.toLowerCase();
            
            const filteredPayments = payments.filter(payment => {
                const matchesSearch = payment.studentName.toLowerCase().includes(searchTerm) || 
                                     payment.mobileNumber.includes(searchTerm);
                const matchesStatus = statusFilter === 'all' || payment.status === statusFilter;
                return matchesSearch && matchesStatus;
            });
            
            paymentsList.innerHTML = '';
            
            if (filteredPayments.length === 0) {
                paymentsList.innerHTML = '<p class="text-gray-500 py-4 text-center">No payments found</p>';
                return;
            }
            
            filteredPayments.forEach(payment => {
                const paymentCard = document.createElement('div');
                paymentCard.className = 'payment-card p-4';
                paymentCard.innerHTML = `
                    <div class="flex flex-col md:flex-row md:items-center md:justify-between">
                        <div class="mb-2 md:mb-0">
                            <h3 class="font-medium">${payment.studentName}</h3>
                            <p class="text-gray-600">${payment.mobileNumber}</p>
                            <p class="text-sm ${payment.status === 'approved' ? 'text-green-600' : 'text-yellow-600'}">
                                ${payment.status.charAt(0).toUpperCase() + payment.status.slice(1)}
                            </p>
                        </div>
                        <div class="flex space-x-2">
                            <button onclick="viewScreenshot('${payment.id}')" class="text-blue-600 hover:text-blue-800 text-sm">
                                View Screenshot
                            </button>
                            ${payment.status === 'pending' ? `
                            <button onclick="approvePayment('${payment.id}')" class="text-green-600 hover:text-green-800 text-sm">
                                Approve
                            </button>
                            ` : ''}
                            <button onclick="deletePayment('${payment.id}')" class="text-red-600 hover:text-red-800 text-sm">
                                Delete
                            </button>
                        </div>
                    </div>
                `;
                paymentsList.appendChild(paymentCard);
            });
        }
        
        // Generate receipt
        function generateReceipt(payment) {
            receiptContent.innerHTML = `
                <div class="space-y-4">
                    <img src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/1df4f548-cbb7-4bbf-81f5-75a217828ac2.png" alt="School van service header with blue background" class="w-full rounded mb-4">
                    <h2 class="text-xl font-bold text-center">PAYMENT RECEIPT</h2>
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <p class="font-medium">Receipt No:</p>
                            <p>${payment.id}</p>
                        </div>
                        <div>
                            <p class="font-medium">Date:</p>
                            <p>${payment.date}</p>
                        </div>
                        <div>
                            <p class="font-medium">Student Name:</p>
                            <p>${payment.studentName}</p>
                        </div>
                        <div>
                            <p class="font-medium">Mobile Number:</p>
                            <p>${payment.mobileNumber}</p>
                        </div>
                        <div>
                            <p class="font-medium">Payment Month:</p>
                            <p>${payment.paymentMonth}</p>
                        </div>
                        <div>
                            <p class="font-medium">Status:</p>
                            <p class="text-green-600">Payment Submitted</p>
                        </div>
                    </div>
                    <div class="mt-6 p-4 bg-gray-100 rounded-lg">
                        <p class="font-medium">Payment Screenshot:</p>
                        <img src="${payment.paymentScreenshot}" alt="Uploaded payment proof" class="mt-2 max-w-full h-auto border rounded">
                    </div>
                    <div class="mt-8 pt-4 border-t text-center text-sm text-gray-500">
                        <p>This is an automatically generated receipt. Please keep it for your records.</p>
                    </div>
                </div>
            `;
        }
        
        // Approve payment
        window.approvePayment = function(id) {
            payments = payments.map(payment => 
                payment.id === parseInt(id) ? {...payment, status: 'approved'} : payment
            );
            savePayments();
            renderPaymentsList();
        };
        
        // Delete payment
        window.deletePayment = function(id) {
            if (confirm('Are you sure you want to delete this payment record?')) {
                payments = payments.filter(payment => payment.id !== parseInt(id));
                savePayments();
                renderPaymentsList();
            }
        };
        
        // View screenshot
        window.viewScreenshot = function(id) {
            const payment = payments.find(p => p.id === parseInt(id));
            if (payment) {
                receiptContent.innerHTML = `
                    <div class="space-y-4">
                        <h3 class="font-bold">${payment.studentName}'s Payment Proof</h3>
                        <img src="${payment.paymentScreenshot}" alt="Payment screenshot for ${payment.studentName}" class="max-w-full h-auto border rounded">
                        <div class="text-sm">
                            <p><span class="font-medium">Mobile:</span> ${payment.mobileNumber}</p>
                            <p><span class="font-medium">Date:</span> ${payment.date}</p>
                            <p><span class="font-medium">Status:</span> <span class="${payment.status === 'approved' ? 'text-green-600' : 'text-yellow-600'}">${payment.status}</span></p>
                        </div>
                    </div>
                `;
                receiptModal.classList.remove('hidden');
            }
        };
        
        // Event listeners
        searchInput.addEventListener('input', (e) => renderPaymentsList(e.target.value));
        filterStatus.addEventListener('change', () => renderPaymentsList(searchInput.value));
        
        printReceipt.addEventListener('click', () => {
            window.print();
        });
        
        closeReceipt.addEventListener('click', () => {
            receiptModal.classList.add('hidden');
        });

        // Close login modal when clicking outside
        document.getElementById('loginModal').addEventListener('click', (e) => {
            if (e.target === document.getElementById('loginModal')) {
                document.getElementById('loginModal').classList.add('hidden');
            }
        });
        
        // Initialize and lo