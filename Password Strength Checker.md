# Password-Strength-Checker
A password strength checker evaluates how secure a password is by testing it against a set of rules: minimum length, uppercase and lowercase letters, numbers, and special characters. A great user experience gives real-time visual feedback as the user types.

Here is an interactive Password Strength Checker simulator. You can type different passwords to see how the strength meter updates, visualizes entropy, and checks off security requirements in real time.<img width="726" height="543" alt="image" src="https://github.com/user-attachments/assets/054314cb-18cd-4e03-96f8-380444ef16af" />
Here is the complete, single-file implementation of the real-time Password Strength Checker. It features visibility toggling, a dynamic strength meter that changes color based on criteria met, and an active checklist for user feedback.

You can save this code as an .html file (e.g., index.html) to run it immediately or copy the respective sections into your existing registration form project.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Password Strength Checker</title>
    <style>
        :root {
            --bg-color: #f4f7f6;
            --card-bg: #ffffff;
            --text-color: #333333;
            --border-color: #dddddd;
            
            /* Strength Colors */
            --weak: #ff4d4d;
            --medium: #ff944d;
            --strong: #ffd11a;
            --excellent: #2ecc71;
            --neutral: #e0e0e0;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }

        .form-container {
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
            width: 100%;
            max-width: 400px;
        }

        h2 {
            margin-top: 0;
            margin-bottom: 20px;
            font-size: 24px;
            text-align: center;
        }

        .input-group {
            position: relative;
            margin-bottom: 15px;
        }

        .input-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            font-size: 14px;
        }

        .password-wrapper {
            position: relative;
            display: flex;
            align-items: center;
        }

        .password-wrapper input {
            width: 100%;
            padding: 12px 40px 12px 12px;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            font-size: 16px;
            outline: none;
            transition: border-color 0.2s;
        }

        .password-wrapper input:focus {
            border-color: #4a90e2;
        }

        .toggle-btn {
            position: absolute;
            right: 12px;
            background: none;
            border: none;
            cursor: pointer;
            font-size: 14px;
            color: #777777;
            user-select: none;
            padding: 0;
        }

        /* Strength Meter Styling */
        .meter-container {
            margin-bottom: 20px;
        }

        .meter-bar {
            height: 6px;
            background-color: var(--neutral);
            border-radius: 3px;
            position: relative;
            overflow: hidden;
            margin-bottom: 8px;
        }

        .meter-progress {
            height: 100%;
            width: 0%;
            background-color: var(--weak);
            transition: width 0.3s ease, background-color 0.3s ease;
        }

        .strength-text {
            font-size: 13px;
            font-weight: 600;
            text-transform: capitalize;
        }

        /* Checklist Requirements */
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }

        .requirement {
            font-size: 14px;
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            color: #777777;
            transition: color 0.2s;
        }

        .requirement::before {
            content: "✕";
            margin-right: 10px;
            color: var(--weak);
            font-weight: bold;
            display: inline-block;
            width: 12px;
        }

        .requirement.valid {
            color: #2c3e50;
        }

        .requirement.valid::before {
            content: "✓";
            color: var(--excellent);
        }

        .submit-btn {
            width: 100%;
            padding: 12px;
            background-color: #4a90e2;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            margin-top: 15px;
            transition: background-color 0.2s;
        }

        .submit-btn:hover {
            background-color: #357abd;
        }
    </style>
</head>
<body>

    <div class="form-container">
        <h2>Create Account</h2>
        <form id="registrationForm" onsubmit="return validateForm(event)">
            
            <div class="input-group">
                <label for="password">Password</label>
                <div class="password-wrapper">
                    <input type="password" id="password" autocomplete="new-password" placeholder="Enter secure password">
                    <button type="button" id="togglePassword" class="toggle-btn">Show</button>
                </div>
            </div>

            <div class="meter-container">
                <div class="meter-bar">
                    <div id="meterProgress" class="meter-progress"></div>
                </div>
                <span id="strengthLabel" class="strength-text" style="color: #777777;">Too Short</span>
            </div>

            <ul class="requirements-list">
                <li id="reqLength" class="requirement">At least 8 characters</li>
                <li id="reqUpper" class="requirement">At least one uppercase letter</li>
                <li id="reqLower" class="requirement">At least one lowercase letter</li>
                <li id="reqNumber" class="requirement">At least one number</li>
                <li id="reqSpecial" class="requirement">At least one special character (@, #, $, etc.)</li>
            </ul>

            <button type="submit" id="submitBtn" class="submit-btn">Register</button>
        </form>
    </div>

    <script>
        const passwordInput = document.getElementById('password');
        const togglePasswordBtn = document.getElementById('togglePassword');
        const meterProgress = document.getElementById('meterProgress');
        const strengthLabel = document.getElementById('strengthLabel');

        // Requirement Elements
        const reqElements = {
            length: document.getElementById('reqLength'),
            upper: document.getElementById('reqUpper'),
            lower: document.getElementById('reqLower'),
            number: document.getElementById('reqNumber'),
            special: document.getElementById('reqSpecial')
        };

        // Strength Configurations
        const strengthConfig = [
            { width: '0%', color: '#777777', text: 'Too Short' },    // 0 rules met
            { width: '25%', color: '#ff4d4d', text: 'Weak' },        // 1 rule met
            { width: '50%', color: '#ff944d', text: 'Medium' },      // 2-3 rules met
            { width: '75%', color: '#ffd11a', text: 'Strong' },      // 4 rules met
            { width: '100%', color: '#2ecc71', text: 'Excellent' }  // All 5 rules met
        ];

        // 1. Toggle Password Visibility
        togglePasswordBtn.addEventListener('click', () => {
            const isPassword = passwordInput.type === 'password';
            passwordInput.type = isPassword ? 'text' : 'password';
            togglePasswordBtn.textContent = isPassword ? 'Hide' : 'Show';
        });

        // 2. Real-time evaluation loop
        passwordInput.addEventListener('input', () => {
            const val = passwordInput.value;

            // Define validation checks
            const checks = {
                length: val.length >= 8,
                upper: /[A-Z]/.test(val),
                lower: /[a-z]/.test(val),
                number: /[0-9]/.test(val),
                special: /[^A-Za-z0-9]/.test(val)
            };

            // Toggle checklist visual states
            let score = 0;
            Object.keys(checks).forEach(key => {
                if (checks[key]) {
                    reqElements[key].classList.add('valid');
                    score++;
                } else {
                    reqElements[key].classList.remove('valid');
                }
            });

            // Handle edge case: if there's text but length rule isn't met, cap or handle state gracefully
            let displayScore = score;
            if (val.length === 0) {
                displayScore = 0;
            } else if (!checks.length) {
                // If it has characters but less than 8, map it down to visual tier 0 or 1
                displayScore = score > 1 ? 1 : score;
            } else {
                // Map out scores 1 to 5 to configuration brackets
                if (score === 5) displayScore = 4;
                else if (score === 4) displayScore = 3;
                else if (score >= 2) displayScore = 2;
                else displayScore = 1;
            }

            // Update DOM UI elements matching configuration arrays
            const currentTier = strengthConfig[displayScore];
            meterProgress.style.width = currentTier.width;
            meterProgress.style.backgroundColor = currentTier.color;
            strengthLabel.textContent = currentTier.text;
            strengthLabel.style.color = currentTier.color;
        });

        // 3. Form submission validation interceptor
        function validateForm(event) {
            const val = passwordInput.value;
            const isFullySecure = val.length >= 8 && /[A-Z]/.test(val) && /[a-z]/.test(val) && /[0-9]/.test(val) && /[^A-Za-z0-9]/.test(val);
            
            if (!isFullySecure) {
                event.preventDefault();
                alert('Please ensure your password meets all safety criteria before registering.');
                return false;
            }
            
            alert('Registration data processed securely!');
            return true;
        }
    </script>
</body>
</html>

Key Structural Highlights:
Off-Flame Architecture: Form submissions validation logic natively checks constraints cleanly via regular expressions before letting a processing state advance.

Component Encapsulation: CSS values utilize customized native runtime variables (:root) allowing you to alter structural branding palettes quickly.

Accessibility Integration: The visibility selector switches field input states smoothly between dynamically protected tokens (type="password") and raw configurations cleanly without field state dropping focus keys.

EXPLANATION
This code builds a secure, interactive password input field using standard web technologies. Here is how the HTML, CSS, and JavaScript work together to make it happen:

1. The Structure (HTML)
The HTML sets up a clean form container with three main parts:

The Input Wrapper: A standard <input> field paired with a <button> that allows users to toggle between showing and hiding their plain-text characters.

The Visual Meter: A structural container (.meter-bar) acting as the track, containing a nested inner div (#meterProgress) that expands like a loading bar as the password gets stronger.

The Requirements Checklist: An unordered list (<ul>) where each list item (<li>) holds a single rule, such as requiring numbers or special characters.

2. The Styling (CSS)
The CSS focuses heavily on clean presentation and instant visual feedback:

CSS Variables (:root): Colors are declared at the top as variables (like --weak: #ff4d4d and --excellent: #2ecc71). This makes it incredibly easy to change the color palette in one place without digging through lines of code.

State Switching: The checklist items rely on a .valid class. By default, items show a red cross (✕). When JavaScript adds the .valid class to a list item, the CSS instantly replaces the cross with a green checkmark (✓) and darkens the text color.

3. The Logic (JavaScript)
The JavaScript runs entirely in the user's browser, handling three distinct responsibilities:
B. Real-Time Testing & Scoring
Every single time a user presses a key inside the input box, an input event listener fires. It runs the text through five validation checks using Regular Expressions (RegEx):

/[A-Z]/.test(val) looks for any uppercase letter.

/[0-9]/.test(val) looks for any number.

/[^A-Za-z0-9]/.test(val) looks for anything that is not a standard letter or number (detecting symbols like @, #, $).

For every check that passes, a score counter increases by 1, and the corresponding list item gets the .valid class.
C. The Grading Scale
Once the final score (0 to 5) is tallied, the script maps that number to a step in the strengthConfig array:
It instantly updates the progress bar's CSS width and background-color, changing it from a short red bar to a full green bar depending on how many rules the user satisfies.

D. The Guardrail (Form Submission)
Finally, the validateForm(event) function prevents users from submitting weak data. If the password fails any of the security criteria, it triggers event.preventDefault(), which halts the form submission and alerts the user to fix their password.
