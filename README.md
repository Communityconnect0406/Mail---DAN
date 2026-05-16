




<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Support Mail</title>

<style>

body{
    margin:0;
    padding:0;
    background:#0f1117;
    font-family:Arial, sans-serif;
    color:white;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
    overflow:hidden;
}

.box{
    width:360px;
    background:#1a1d26;
    padding:25px;
    border-radius:12px;
    box-shadow:0 0 20px rgba(0,0,0,0.5);
}

h2{
    text-align:center;
    margin-bottom:20px;
}

input, textarea{
    width:100%;
    padding:12px;
    margin-top:10px;
    border:none;
    border-radius:8px;
    background:#2a2f3b;
    color:white;
    box-sizing:border-box;
}

textarea{
    resize:none;
}

button{
    width:100%;
    padding:12px;
    margin-top:15px;
    border:none;
    border-radius:8px;
    background:#5865F2;
    color:white;
    font-size:16px;
    cursor:pointer;
}

button:hover{
    background:#4752c4;
}

.status{
    margin-top:12px;
    text-align:center;
}

/* BLOCK SCREEN */

#blockedScreen{
    position:fixed;
    top:0;
    left:0;
    width:100%;
    height:100%;
    background:black;
    color:red;
    display:none;
    justify-content:center;
    align-items:center;
    flex-direction:column;
    z-index:9999;
    text-align:center;
}

#blockedScreen h1{
    font-size:45px;
    margin-bottom:10px;
}

#blockedScreen p{
    color:white;
    font-size:20px;
    margin-bottom:25px;
}

#unlockButton{
    width:220px;
}

</style>
</head>
<body>

<div class="box" id="formBox">

    <h2>Support Mail</h2>

    <input type="text" id="discordUser" placeholder="Discord Username">

    <input type="text" id="discordID" placeholder="Discord ID">

    <textarea id="issue" rows="5" placeholder="Describe your issue"></textarea>

    <button onclick="sendMail()">Send</button>

    <div class="status" id="status"></div>

</div>

<!-- BLOCKED SCREEN -->

<div id="blockedScreen">

    <h1>ACCESS BLOCKED</h1>

    <p>Due to your actions, you have been blocked.</p>

    <button id="unlockButton" onclick="adminUnlock()">
        Admin Unlock
    </button>

</div>

<script>

// WEBHOOKS

const supportWebhook =
"https://discord.com/api/webhooks/1505255569689415781/Oh54IN01hApGf0OSLHmQBNkEfFDzdRMzh2vhANvI2qko8ZMhgwSYD7oBJz08FwtTkz_D";

const flaggedWebhook =
"https://discord.com/api/webhooks/1505256514003406898/e1FoqiUA9Ev5IFkX-eWU-Qx0lvDeoAEPQKTO1VjdRmwV_W_vnxHieWD4X7ujBiRREEdp";

// BAD WORD FILTER

const blockedWords = [
    "fuck",
    "shit",
    "bitch",
    "asshole",
    "retard",
    "kys",
    "kill yourself"
];

// GENERATE TICKET ID

function generateTicketID(){
    return Math.floor(1000 + Math.random() * 9000);
}

// CHECK BAD WORDS

function containsBlockedWords(text){

    text = text.toLowerCase();

    return blockedWords.some(word => text.includes(word));

}

// SHOW BLOCK SCREEN

function showBlockedScreen(){

    document.getElementById("blockedScreen").style.display = "flex";

    document.getElementById("formBox").style.display = "none";

    localStorage.setItem("blocked","true");

}

// ADMIN UNLOCK

function adminUnlock(){

    const code = prompt("Enter Admin Code");

    if(code === "4242"){

        localStorage.removeItem("blocked");

        document.getElementById("blockedScreen").style.display = "none";

        document.getElementById("formBox").style.display = "block";

        alert("Block Removed");

    }else{

        alert("Wrong Code");

    }

}

// IF USER IS BLOCKED

if(localStorage.getItem("blocked") === "true"){

    showBlockedScreen();

}

// SEND WEBHOOK

async function sendWebhook(url, data){

    return fetch(url,{
        method:"POST",
        headers:{
            "Content-Type":"application/json"
        },
        body:JSON.stringify(data)
    });

}

// SEND MAIL

async function sendMail(){

    const user =
    document.getElementById("discordUser").value.trim();

    const discordID =
    document.getElementById("discordID").value.trim();

    const issue =
    document.getElementById("issue").value.trim();

    const status =
    document.getElementById("status");

    // CHECK EMPTY

    if(!user || !discordID || !issue){

        status.innerHTML = "Fill in all fields.";

        status.style.color = "red";

        return;

    }

    const ticketID = generateTicketID();

    // BAD WORD DETECTED

    if(
        containsBlockedWords(user) ||
        containsBlockedWords(issue)
    ){

        const flaggedData = {

            content:"⚠️ Inappropriate Message Detected",

            embeds:[

                {

                    title:"Blocked Form Attempt",

                    color:16711680,

                    fields:[

                        {
                            name:"Ticket ID",
                            value:"#"+ticketID,
                            inline:true
                        },

                        {
                            name:"Discord User",
                            value:user,
                            inline:true
                        },

                        {
                            name:"Discord ID",
                            value:discordID
                        },

                        {
                            name:"Message",
                            value:issue
                        }

                    ],

                    timestamp:new Date()

                }

            ]

        };

        await sendWebhook(flaggedWebhook, flaggedData);

        alert(
            "You used inappropriate language and have been blocked."
        );

        showBlockedScreen();

        return;

    }

    // NORMAL MESSAGE

    const data = {

        content:"<@1505247831663841423>",

        embeds:[

            {

                title:"📩 New Support Mail",

                color:5793266,

                fields:[

                    {
                        name:"Ticket ID",
                        value:"#"+ticketID,
                        inline:true
                    },

                    {
                        name:"Discord User",
                        value:user,
                        inline:true
                    },

                    {
                        name:"Discord ID",
                        value:discordID
                    },

                    {
                        name:"Issue",
                        value:issue
                    }

                ],

                timestamp:new Date()

            }

        ]

    };

    try{

        const response =
        await sendWebhook(supportWebhook, data);

        if(response.ok){

            status.innerHTML =
            "Mail Sent! Ticket #"+ticketID;

            status.style.color = "lime";

            document.getElementById("discordUser").value = "";

            document.getElementById("discordID").value = "";

            document.getElementById("issue").value = "";

        }else{

            status.innerHTML = "Failed to send.";

            status.style.color = "red";

        }

    }catch(error){

        status.innerHTML = "Error sending.";

        status.style.color = "red";

    }

}

</script>

</body>
</html>
