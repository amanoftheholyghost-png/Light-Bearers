<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Church Member Records</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: Arial, sans-serif; background: #f4f6f9; padding: 20px; }
    .container { max-width: 1000px; margin: auto; background: white; padding: 25px;
                 border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
    h1 { color: #2c3e50; margin-bottom: 20px; text-align: center; }
    .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
    input, select { padding: 10px; border: 1px solid #ccc; border-radius: 6px; font-size: 14px; }
    button { padding: 10px 18px; border: none; border-radius: 6px; cursor: pointer;
             font-size: 14px; font-weight: bold; }
    .btn-add { background: #27ae60; color: white; grid-column: span 2; }
    .btn-add:hover { background: #219150; }
    .btn-del { background: #e74c3c; color: white; padding: 6px 12px; font-size: 12px; }
    .btn-edit { background: #3498db; color: white; padding: 6px 12px; font-size: 12px; margin-right: 5px; }
    table { width: 100%; border-collapse: collapse; margin-top: 15px; }
    th, td { padding: 10px; text-align: left; border-bottom: 1px solid #eee; font-size: 14px; }
    th { background: #34495e; color: white; }
    tr:hover { background: #f9f9f9; }
    .search { width: 100%; margin-bottom: 15px; }
    .empty { text-align: center; color: #999; padding: 30px; }
  </style>
</head>
<body>
  <div class="container">
    <h1>⛪ Church Member Records</h1>

    <div class="form-grid">
      <input id="name" placeholder="Full Name *">
      <input id="phone" placeholder="Phone Number">
      <input id="email" placeholder="Email">
      <input id="birthday" type="date" placeholder="Birthday">
      <input id="address" placeholder="Home Address">
      <select id="gender">
        <option value="">Select Gender</option>
        <option>Male</option>
        <option>Female</option>
      </select>
      <input id="joinDate" type="date" title="Join Date">
      <input id="ministry" placeholder="Ministry / Department">
      <button class="btn-add" onclick="addMember()">➕ Add Member</button>
    </div>

    <input class="search" id="search" placeholder="🔍 Search by name, phone, or ministry..." oninput="renderMembers()">

    <table>
      <thead>
        <tr>
          <th>Name</th><th>Phone</th><th>Email</th><th>Birthday</th>
          <th>Ministry</th><th>Joined</th><th>Actions</th>
        </tr>
      </thead>
      <tbody id="memberList"></tbody>
    </table>
  </div>

  <script>
    let members = JSON.parse(localStorage.getItem('churchMembers') || '[]');
    let editIndex = -1;

    function save() {
      localStorage.setItem('churchMembers', JSON.stringify(members));
    }

    function addMember() {
      const name = document.getElementById('name').value.trim();
      if (!name) { alert('Please enter a name'); return; }

      const member = {
        name,
        phone: document.getElementById('phone').value,
        email: document.getElementById('email').value,
        birthday: document.getElementById('birthday').value,
        address: document.getElementById('address').value,
        gender: document.getElementById('gender').value,
        joinDate: document.getElementById('joinDate').value,
        ministry: document.getElementById('ministry').value
      };

      if (editIndex === -1) {
        members.push(member);
      } else {
        members[editIndex] = member;
        editIndex = -1;
      }

      save();
      clearForm();
      renderMembers();
    }

    function editMember(i) {
      const m = members[i];
      document.getElementById('name').value = m.name;
      document.getElementById('phone').value = m.phone;
      document.getElementById('email').value = m.email;
      document.getElementById('birthday').value = m.birthday;
      document.getElementById('address').value = m.address;
      document.getElementById('gender').value = m.gender;
      document.getElementById('joinDate').value = m.joinDate;
      document.getElementById('ministry').value = m.ministry;
      editIndex = i;
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function deleteMember(i) {
      if (confirm('Delete this member?')) {
        members.splice(i, 1);
        save();
        renderMembers();
      }
    }

    function clearForm() {
      ['name','phone','email','birthday','address','gender','joinDate','ministry']
        .forEach(id => document.getElementById(id).value = '');
    }

    function renderMembers() {
      const query = document.getElementById('search').value.toLowerCase();
      const list = document.getElementById('memberList');
      const filtered = members
        .map((m, i) => ({ ...m, i }))
        .filter(m =>
          m.name.toLowerCase().includes(query) ||
          m.phone.includes(query) ||
          m.ministry.toLowerCase().includes(query)
        );

      if (!filtered.length) {
        list.innerHTML = '<tr><td colspan="7" class="empty">No members found</td></tr>';
        return;
      }

      list.innerHTML = filtered.map(m => `
        <tr>
          <td>${escapeHtml(m.name)}</td>
          <td>${escapeHtml(m.phone)}</td>
          <td>${escapeHtml(m.email)}</td>
          <td>${m.birthday || '-'}</td>
          <td>${escapeHtml(m.ministry)}</td>
          <td>${m.joinDate || '-'}</td>
          <td>
            <button class="btn-edit" onclick="editMember(${m.i})">Edit</button>
            <button class="btn-del" onclick="deleteMember(${m.i})">Delete</button>
          </td>
        </tr>
      `).join('');
    }

    function escapeHtml(s) {
      return (s || '').replace(/[&<>"']/g, c =>
        ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
    }

    renderMembers();
  </script>
</body>
</html>
