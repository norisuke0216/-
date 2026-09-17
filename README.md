<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>文化祭 写真受け取り</title>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

<style>
body {
  font-family: sans-serif;
  max-width: 500px;
  margin: auto;
  padding: 25px;
  background: #f5f5f5;
}

.box {
  background: white;
  padding: 25px;
  border-radius: 15px;
}

input, button {
  width: 100%;
  box-sizing: border-box;
  padding: 13px;
  margin: 7px 0;
  font-size: 16px;
}

button {
  background: #222;
  color: white;
  border: none;
  border-radius: 8px;
}

img {
  width: 100%;
  margin-top: 15px;
  border-radius: 10px;
}
</style>
</head>

<body>

<div class="box">

<h1>文化祭 写真受け取り</h1>

<p>受付番号とパスワードを入力してください。</p>

<input id="number" placeholder="受付番号（例：001）">
<input id="password" type="password" placeholder="パスワード">

<button onclick="login()">ログイン</button>

<p id="message"></p>

<div id="photos"></div>

<button id="logout" onclick="logout()" style="display:none;">
ログアウト
</button>

</div>

<script>

const SUPABASE_URL =
"https://lbgtsgidsllidgjiucsy.supabase.co";

const SUPABASE_PUBLISHABLE_KEY =
"sb_publishable_BY6eGooGtlg21KCrVHX0gw_aoNJOBup";

const supabaseClient =
supabase.createClient(
  SUPABASE_URL,
  SUPABASE_PUBLISHABLE_KEY
);


async function login() {

  const number =
    document.getElementById("number").value.trim();

  const password =
    document.getElementById("password").value;

  const message =
    document.getElementById("message");

  if (!number || !password) {
    message.textContent =
      "受付番号とパスワードを入力してください。";
    return;
  }

  message.textContent = "ログイン中…";

  const { error } =
    await supabaseClient.auth.signInWithPassword({

      email: number + "@festival.local",

      password: password

    });

  if (error) {

    message.textContent =
      "受付番号またはパスワードが違います。";

    return;
  }

  document.getElementById("number").style.display = "none";
  document.getElementById("password").style.display = "none";

  document.querySelector("button").style.display = "none";

  document.getElementById("logout").style.display = "block";

  await showPhotos();
}


async function showPhotos() {

  const message =
    document.getElementById("message");

  const photos =
    document.getElementById("photos");

  const { data: { user } } =
    await supabaseClient.auth.getUser();

  if (!user) return;

  const { data, error } =
    await supabaseClient
      .storage
      .from("Photos")
      .list(user.id);

  if (error) {

    message.textContent =
      "写真を読み込めませんでした。";

    return;
  }

  photos.innerHTML = "";

  for (const file of data) {

    if (!file.name) continue;

    const { data: fileData } =
      await supabaseClient
        .storage
        .from("Photos")
        .download(
          user.id + "/" + file.name
        );

    if (!fileData) continue;

    const img =
      document.createElement("img");

    img.src =
      URL.createObjectURL(fileData);

    photos.appendChild(img);
  }

  message.textContent =
    data.length ? "写真" : "写真はまだありません。";
}


async function logout() {

  await supabaseClient.auth.signOut();

  location.reload();

}

</script>

</body>
</html>
