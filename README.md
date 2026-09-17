<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>文化祭 写真受け取り</title>

  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

  <style>
    body {
      font-family: sans-serif;
      max-width: 500px;
      margin: 0 auto;
      padding: 25px;
      background: #f5f5f5;
    }

    .box {
      background: white;
      padding: 25px;
      border-radius: 15px;
    }

    input,
    button {
      width: 100%;
      box-sizing: border-box;
      padding: 13px;
      margin: 7px 0;
      font-size: 16px;
    }

    input {
      border: 1px solid #ccc;
      border-radius: 8px;
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

    #logout {
      display: none;
      background: #777;
    }

    #message {
      min-height: 24px;
    }
  </style>
</head>

<body>

  <div class="box">

    <h1>文化祭 写真受け取り</h1>

    <p>受付番号とパスワードを入力してください。</p>

    <div id="loginArea">
      <input
        id="number"
        type="text"
        placeholder="受付番号（例：001）"
        inputmode="numeric"
      >

      <input
        id="password"
        type="password"
        placeholder="パスワード"
      >

      <button onclick="login()">ログイン</button>
    </div>

    <p id="message"></p>

    <div id="photos"></div>

    <button id="logout" onclick="logout()">
      ログアウト
    </button>

  </div>

  <script>

    /* SupabaseのプロジェクトURL */
    const SUPABASE_URL =
      "https://lbgtsgidsllidgjiucsy.supabase.co";

    /*
      sb_publishable_BY6eGooGtlg21KCrVHX0gw_aoNJOBup.";
    */
    const SUPABASE_PUBLISHABLE_KEY =
      "sb_publishable_BY6eGooGtlg21KCrVHX0gw_aoNJOBup";

    const supabaseClient =
      supabase.createClient(
        SUPABASE_URL,
        SUPABASE_PUBLISHABLE_KEY
      );


    /* ログイン */
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


      document.getElementById("loginArea")
        .style.display = "none";

      document.getElementById("logout")
        .style.display = "block";

      await showPhotos();
    }


    /* 写真を表示 */
    async function showPhotos() {

      const message =
        document.getElementById("message");

      const photos =
        document.getElementById("photos");


      const {
        data: { user }
      } =
        await supabaseClient.auth.getUser();


      if (!user) {

        message.textContent =
          "ログインしてください。";

        return;
      }


      /*
        ログインしているユーザー自身の
        UIDフォルダだけを読み込む
      */
      const { data, error } =
        await supabaseClient
          .storage
          .from("Photos")
          .list(user.id, {
            limit: 100
          });


      if (error) {

        message.textContent =
          "写真を読み込めませんでした。";

        return;
      }


      photos.innerHTML = "";


      if (!data || data.length === 0) {

        message.textContent =
          "写真はまだありません。";

        return;
      }


      for (const file of data) {

        if (!file.name) continue;


        const {
          data: fileData,
          error: downloadError
        } =
          await supabaseClient
            .storage
            .from("Photos")
            .download(
              user.id + "/" + file.name
            );


        if (downloadError) continue;


        const img =
          document.createElement("img");

        img.src =
          URL.createObjectURL(fileData);

        img.alt = "受け取った写真";

        photos.appendChild(img);
      }


      message.textContent = "写真";
    }


    /* ログアウト */
    async function logout() {

      await supabaseClient.auth.signOut();

      location.reload();
    }

  </script>

</body>
</html>
