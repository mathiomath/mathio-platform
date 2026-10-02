const STORAGE_KEY = "mathioState";

const authContainer = document.getElementById("auth-container");
const dashboardContainer = document.getElementById("dashboard-container");
const loginForm = document.getElementById("login-form");
const logoutBtn = document.getElementById("logout-btn");
const usernameInput = document.getElementById("username");
const passwordInput = document.getElementById("password");
const classSelect = document.getElementById("class-select");
const displayName = document.getElementById("display-name");
const displayClass = document.getElementById("display-class");
const puanCount = document.getElementById("puan-count");
const toast = document.getElementById("toast");

function readState() {
  try {
    const rawState = localStorage.getItem(STORAGE_KEY);
    return rawState ? JSON.parse(rawState) : {};
  } catch {
    return {};
  }
}

function writeState(state) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function showToast(message) {
  toast.textContent = message;
  toast.classList.add("show");
  clearTimeout(showToast.timeoutId);
  showToast.timeoutId = setTimeout(() => {
    toast.classList.remove("show");
  }, 1800);
}

function renderDashboard(username, selectedClass, score) {
  displayName.textContent = username;
  displayClass.textContent = `${selectedClass} Sınıfı`;
  puanCount.textContent = score;

  authContainer.style.display = "none";
  dashboardContainer.style.display = "block";
}

function loadSession() {
  const state = readState();
  if (state.isLoggedIn && state.username && state.className) {
    renderDashboard(state.username, state.className, state.score || 0);
  }
}

function handleLogin(event) {
  event.preventDefault();

  const username = usernameInput.value.trim();
  const password = passwordInput.value.trim();
  const selectedClass = classSelect.value;

  if (!username || !password) {
    showToast("Kullanıcı adı ve şifre gerekli!");
    return;
  }

  const state = {
    isLoggedIn: true,
    username,
    className: selectedClass,
    score: 0,
  };

  writeState(state);
  renderDashboard(username, selectedClass, 0);
  showToast("Hoş geldin, Mathio seni bekliyordu!");
}

function handleLogout() {
  writeState({});
  authContainer.style.display = "block";
  dashboardContainer.style.display = "none";
  loginForm.reset();
  showToast("Çıkış yapıldı.");
}

function markAnswerButtons(card, selectedButton, isCorrect) {
  const options = card.querySelectorAll(".quiz-option");
  options.forEach((option) => {
    option.disabled = true;
    if (option === selectedButton && isCorrect) {
      option.classList.add("correct");
    }
    if (option === selectedButton && !isCorrect) {
      option.classList.add("wrong");
    }
    if (option.dataset.correct === "true" && !selectedButton.dataset.correct) {
      option.classList.add("correct");
    }
  });
}

function handleAnswer(event) {
  const selectedButton = event.target.closest(".quiz-option");
  if (!selectedButton || selectedButton.disabled) return;

  const card = selectedButton.closest(".topic-card");
  const isCorrect = selectedButton.dataset.correct === "true";
  const state = readState();

  if (!state.isLoggedIn) return;

  if (isCorrect) {
    state.score = (Number(state.score) || 0) + 10;
    writeState(state);
    puanCount.textContent = state.score;
    showToast("Tebrikler! +10 Pati Puan");
  } else {
    showToast("Tavsiye: Mathio diyor ki, tekrar dene!");
  }

  markAnswerButtons(card, selectedButton, isCorrect);
}

loginForm.addEventListener("submit", handleLogin);
logoutBtn.addEventListener("click", handleLogout);
document.addEventListener("click", function (event) {
  if (event.target.classList.contains("quiz-option")) {
    handleAnswer(event);
  }
});

loadSession();

window.addEventListener("beforeunload", () => {
  const state = readState();
  if (!state.isLoggedIn) return;
  writeState({ ...state, score: Number(state.score) || 0 });
});

