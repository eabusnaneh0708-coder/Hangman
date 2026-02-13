<!DOCTYPE html>
<html>
<head>
    <title>Hangman - Lunch</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f4f4f4;
            margin-top: 50px;
        }
        h1 {
            color: #333;
        }
        #word {
            font-size: 40px;
            letter-spacing: 10px;
            margin: 20px;
        }
        #letters button {
            margin: 5px;
            padding: 10px 15px;
            font-size: 16px;
            cursor: pointer;
        }
        #message {
            font-size: 20px;
            margin-top: 20px;
            font-weight: bold;
        }
    </style>
</head>
<body>

<h1>🍴 Hangman Game</h1>
<p>Guess the word!</p>

<div id="word"></div>
<div id="letters"></div>
<div id="message"></div>

<script>
    const word = "lunch";
    let guessed = [];
    let wrong = 0;
    const maxWrong = 6;

    const wordDiv = document.getElementById("word");
    const lettersDiv = document.getElementById("letters");
    const messageDiv = document.getElementById("message");

    function displayWord() {
        wordDiv.innerHTML = word
            .split("")
            .map(letter => guessed.includes(letter) ? letter : "_")
            .join(" ");
    }

    function createLetters() {
        const alphabet = "abcdefghijklmnopqrstuvwxyz";
        alphabet.split("").forEach(letter => {
            const button = document.createElement("button");
            button.innerText = letter;
            button.onclick = () => guessLetter(letter, button);
            lettersDiv.appendChild(button);
        });
    }

    function guessLetter(letter, button) {
        button.disabled = true;

        if (word.includes(letter)) {
            guessed.push(letter);
            displayWord();
            checkWin();
        } else {
            wrong++;
            if (wrong >= maxWrong) {
                messageDiv.innerText = "💀 Game Over! The word was LUNCH.";
                disableAll();
            }
        }
    }

    function checkWin() {
        if (word.split("").every(letter => guessed.includes(letter))) {
            messageDiv.innerText = "🎉 You guessed it! The word is LUNCH!";
            disableAll();
        }
    }

    function disableAll() {
        const buttons = document.querySelectorAll("button");
        buttons.forEach(btn => btn.disabled = true);
    }

    displayWord();
    createLetters();
</script>

</body>
</html>
