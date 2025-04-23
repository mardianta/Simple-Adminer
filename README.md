# Simple-Adminer
```
<?php
$host = $_GET['host'] ?? '';
$user = $_GET['user'] ?? '';
$pass = $_GET['pass'] ?? '';
$db   = $_GET['db'] ?? '';
$table = $_GET['table'] ?? '';
$sql = $_POST['sql'] ?? '';

$mysqli = null;
$error_message = '';

if ($host && $user) {
    $mysqli = new mysqli($host, $user, $pass, $db ?: null);
    if ($mysqli->connect_error) {
        $error_message = $mysqli->connect_error;
    }
}

function getDatabases($mysqli) {
    $res = $mysqli->query("SHOW DATABASES");
    return $res ? array_column($res->fetch_all(), 0) : [];
}

function getTables($mysqli) {
    $res = $mysqli->query("SHOW TABLES");
    return $res ? array_column($res->fetch_all(), 0) : [];
}

$tableData = null;
if ($mysqli && $table) {
    $tableData = $mysqli->query("SELECT * FROM `$table`");
}

$sqlResult = null;
if ($mysqli && $sql) {
    $sqlResult = $mysqli->query($sql);
}
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Simple DB Viewer</title>
    <style>
        body { font-family: sans-serif; margin: 20px; }
        input, select, textarea { margin: 6px 0; width: 100%; padding: 6px; }
        table { border-collapse: collapse; width: 100%; margin-top: 20px; }
        th, td { border: 1px solid #ccc; padding: 6px; }
    </style>
</head>
<body>
<h2>Simple DB Viewer + SQL Executor</h2>

<?php if (!$mysqli || $error_message): ?>
    <form method="get">
        <label>Host:</label><input name="host" required value="<?= htmlspecialchars($host) ?>">
        <label>User:</label><input name="user" required value="<?= htmlspecialchars($user) ?>">
        <label>Password:</label><input name="pass" type="password" value="<?= htmlspecialchars($pass) ?>">
        <label>Database (opsional):</label><input name="db" value="<?= htmlspecialchars($db) ?>">
        <button type="submit">Connect</button>
    </form>
    <?php if ($error_message): ?><p style="color:red;">Error: <?= htmlspecialchars($error_message) ?></p><?php endif; ?>
<?php else: ?>
    <form method="get">
        <input type="hidden" name="host" value="<?= $host ?>">
        <input type="hidden" name="user" value="<?= $user ?>">
        <input type="hidden" name="pass" value="<?= $pass ?>">
        <label>Database:
            <select name="db" onchange="this.form.submit()">
                <?php foreach (getDatabases($mysqli) as $d): ?>
                    <option <?= $d === $db ? 'selected' : '' ?> value="<?= $d ?>"><?= $d ?></option>
                <?php endforeach; ?>
            </select>
        </label>
        <?php if ($db): ?>
            <label>Tabel:
                <select name="table" onchange="this.form.submit()">
                    <?php foreach (getTables($mysqli) as $t): ?>
                        <option <?= $t === $table ? 'selected' : '' ?> value="<?= $t ?>"><?= $t ?></option>
                    <?php endforeach; ?>
                </select>
            </label>
        <?php endif; ?>
    </form>

    <?php if ($tableData): ?>
        <h3>Data dari Tabel: <?= htmlspecialchars($table) ?></h3>
        <table>
            <thead>
                <tr>
                    <?php foreach ($tableData->fetch_fields() as $field): ?>
                        <th><?= $field->name ?></th>
                    <?php endforeach; ?>
                </tr>
            </thead>
            <tbody>
                <?php while ($row = $tableData->fetch_assoc()): ?>
                    <tr>
                        <?php foreach ($row as $cell): ?>
                            <td><?= htmlspecialchars((string)$cell) ?></td>
                        <?php endforeach; ?>
                    </tr>
                <?php endwhile; ?>
            </tbody>
        </table>
    <?php endif; ?>

    <h3>SQL Manual Executor</h3>
    <form method="post">
        <input type="hidden" name="host" value="<?= $host ?>">
        <input type="hidden" name="user" value="<?= $user ?>">
        <input type="hidden" name="pass" value="<?= $pass ?>">
        <input type="hidden" name="db" value="<?= $db ?>">
        <textarea name="sql" rows="5" placeholder="Ketik perintah SQL di sini..."><?= htmlspecialchars($sql) ?></textarea>
        <button type="submit">Eksekusi</button>
    </form>

    <?php if ($sql): ?>
        <h4>Hasil SQL:</h4>
        <?php if ($sqlResult instanceof mysqli_result): ?>
            <table>
                <thead>
                    <tr>
                        <?php foreach ($sqlResult->fetch_fields() as $field): ?>
                            <th><?= $field->name ?></th>
                        <?php endforeach; ?>
                    </tr>
                </thead>
                <tbody>
                    <?php while ($row = $sqlResult->fetch_assoc()): ?>
                        <tr>
                            <?php foreach ($row as $cell): ?>
                                <td><?= htmlspecialchars((string)$cell) ?></td>
                            <?php endforeach; ?>
                        </tr>
                    <?php endwhile; ?>
                </tbody>
            </table>
        <?php elseif ($sqlResult === true): ?>
            <p>Query berhasil dijalankan.</p>
        <?php else: ?>
            <p style="color:red;">Error: <?= $mysqli->error ?></p>
        <?php endif; ?>
    <?php endif; ?>
<?php endif; ?>
<h4>Cheat Sheet SQL Dasar</h4>
    <ul>
        <li><strong>SELECT</strong>: <code>SELECT * FROM nama_tabel;</code></li>
        <li><strong>INSERT</strong>: <code>INSERT INTO nama_tabel (kolom1, kolom2) VALUES ('nilai1', 'nilai2');</code></li>
        <li><strong>UPDATE</strong>: <code>UPDATE nama_tabel SET kolom1 = 'nilai_baru' WHERE id = 1;</code></li>
        <li><strong>DELETE</strong>: <code>DELETE FROM nama_tabel WHERE id = 1;</code></li>
        <li><strong>Custom</strong>: <code>SHOW TABLES;</code>, <code>DESCRIBE nama_tabel;</code></li>
    </ul>

</body>
</html>

```
