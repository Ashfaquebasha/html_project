<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User List</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/antd/dist/antd.min.css">
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
        }
        .container {
            text-align: center;
            background-color: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 80%;
            max-width: 600px;
        }
        .error {
            color: red;
            margin-top: 10px;
        }
        .ant-btn-delete {
            background-color: #f5222d;
            color: #fff;
            border: none;
            border-radius: 4px;
            padding: 6px 12px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .ant-btn-delete:hover {
            background-color: #cf1322;
        }
        .ant-btn-edit {
            background-color: #1890ff;
            color: #fff;
            border: none;
            border-radius: 4px;
            padding: 6px 12px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .ant-btn-edit:hover {
            background-color: #096dd9;
        }
        .ant-btn-confirm {
            background-color: #52c41a;
            color: #fff;
            border: none;
            border-radius: 4px;
            padding: 6px 12px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .ant-btn-confirm:hover {
            background-color: #389e0d;
        }
        .modal {
            display: none;
            position: fixed;
            z-index: 1;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.4);
        }
        .modal-content {
            background-color: #fefefe;
            margin: 15% auto;
            padding: 20px;
            border: 1px solid #888;
            width: 80%;
            max-width: 400px;
            border-radius: 8px;
            text-align: center;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>User List</h2>

        <form id="userForm">
            <label for="name" style="display: block;">Name:</label>
            <input type="text" id="name" name="name" class="ant-input">
            <label for="email" style="display: block;">Email:</label>
            <input type="email" id="email" name="email" class="ant-input">
            <input type="hidden" id="userId" name="userId">
            <button type="button" class="ant-btn ant-btn-primary" onclick="addOrUpdateUser()">Add User</button>
            <div id="error" class="error"></div>
        </form>
        <div id="userTable"></div>
        <div id="confirmationModal" class="modal">
            <div class="modal-content">
                <p>Are you sure you want to delete this user?</p>
                <button class="ant-btn ant-btn-confirm" onclick="confirmDelete()">Confirm</button>
                <button class="ant-btn" onclick="closeModal()">Cancel</button>
            </div>
        </div>
    </div>
    <script>
        let userId = 1;
        const users = [];
        let editingUserId = null;
        function addOrUpdateUser() {
            const name = document.getElementById('name').value.trim();
            const email = document.getElementById('email').value.trim();
            const error = document.getElementById('error');
            const userIdInput = document.getElementById('userId').value;
            if (name === '' || email === '') {
                error.textContent = 'Both name and email fields are required.';
                return;
            }
            const emailPattern = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}$/;
            if (!emailPattern.test(email)) {
                error.textContent = 'Please enter a valid email address.';
                return;
            }

            if (userIdInput) {
                updateUser(userIdInput, name, email);
            } else {
                addUser(name, email);
            }
            document.getElementById('name').value = '';
            document.getElementById('email').value = '';
            document.getElementById('userId').value = '';
        }

        function addUser(name, email) {
            const user = {
                id: userId,
                name: name,
                email: email
            };
            users.push(user);
            userId++;
            renderUsers();
        }

        function updateUser(id, name, email) {
            const index = users.findIndex(user => user.id === parseInt(id));
            if (index !== -1) {
                users[index].name = name;
                users[index].email = email;
                renderUsers();
            }
        }

        function renderUsers() {
            const tableContainer = document.getElementById('userTable');
            tableContainer.innerHTML = '';

            const table = document.createElement('table');
            table.classList.add('ant-table', 'ant-table-bordered', 'ant-table-striped', 'ant-table-hover');
            const thead = document.createElement('thead');
            const tbody = document.createElement('tbody');
            const headerRow = document.createElement('tr');
            const headers = ['ID', 'Name', 'Email', 'Actions'];
            headers.forEach(headerText => {
                const th = document.createElement('th');
                th.textContent = headerText;
                headerRow.appendChild(th);
            });
            thead.appendChild(headerRow);
            table.appendChild(thead);
            users.forEach(user => {
                const row = document.createElement('tr');
                const idCell = document.createElement('td');
                idCell.textContent = user.id;
                const nameCell = document.createElement('td');
                nameCell.textContent = user.name;
                const emailCell = document.createElement('td');
                emailCell.textContent = user.email;
                const actionsCell = document.createElement('td');
                const editButton = document.createElement('button');
                editButton.textContent = 'Edit';
                editButton.classList.add('ant-btn', 'ant-btn-edit');
                editButton.onclick = function() {
                    editUser(user.id, user.name, user.email);
                };
                actionsCell.appendChild(editButton);
                const deleteButton = document.createElement('button');
                deleteButton.textContent = 'Delete';
                deleteButton.classList.add('ant-btn', 'ant-btn-delete');
                deleteButton.onclick = function() {
                    openDeleteConfirmation(user.id);
                };
                actionsCell.appendChild(deleteButton);
                row.appendChild(idCell);
                row.appendChild(nameCell);
                row.appendChild(emailCell);
                row.appendChild(actionsCell);
                tbody.appendChild(row);
            });
            table.appendChild(tbody);
            tableContainer.appendChild(table);
        }

        function editUser(id, name, email) {
            document.getElementById('name').value = name;
            document.getElementById('email').value = email;
            document.getElementById('userId').value = id;
            editingUserId = id;
        }

        function deleteUser(id) {
            const index = users.findIndex(user => user.id === id);
            if (index !== -1) {
                users.splice(index, 1);
                closeModal();
                renderUsers();
            }
        }

        function openDeleteConfirmation(id) {
            editingUserId = id;
            const modal = document.getElementById('confirmationModal');
            modal.style.display = 'block';
        }

        function confirmDelete() {
            if (editingUserId) {
                deleteUser(editingUserId);
            }
        }

        function closeModal() {
            const modal = document.getElementById('confirmationModal');
            modal.style.display = 'none';
            editingUserId = null;
        }
        window.onclick = function(event) {
            const modal = document.getElementById('confirmationModal');
            if (event.target === modal) {
                closeModal();
            }
        };
    </script>

</body>
</html>
