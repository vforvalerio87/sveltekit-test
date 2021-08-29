<script lang="ts">
  import axios from "axios"

  const REQUIRED_VALIDITY_MESSAGE = "Required"
  const baseUrl = "https://api-staging.holyart.io/v1/users/login"
  let user, error, formValid, promise, emailInput, passwordInput

  function checkFormValidity (ev) { 
    if (ev.target.validity.valid) ev.target.parentElement.removeAttribute("data-validity-message")
    else if (ev.target.hasAttribute("required") && ev.target.value === "") ev.target.parentElement.setAttribute("data-validity-message", REQUIRED_VALIDITY_MESSAGE)
    else ev.target.parentElement.setAttribute("data-validity-message", "Invalid " + ev.target.getAttribute("type"))

    formValid = ev.target.form.checkValidity() 
  }

  async function login () {
    return axios.post(baseUrl, {
      "domainCode": "it", "email": emailInput.value, "password": passwordInput.value
    })
    .then(res => {
      error = null
      formValid = false
      user = res.data
    })
    .catch(err => {
      const detailedResponse =  err?.response?.data?.error?.message &&
                                err.response.data.error.details &&
                                Array.isArray(err.response.data.error.details) ?
                                  `${err.response.data.error.message}: ${err.response.data.error.details.map(field => field.message).join(", ")}` :
                                  undefined

      error = detailedResponse ||
              err?.response?.data?.error?.details?.message ||
              err?.response?.data?.error?.message || 
              err?.message ||
              "There was an error with your request"
    })
  }

  function logout() { user = null }
</script>

{#if user == null}
  <form novalidate action={baseUrl} method="POST">
    <label data-validity-message={REQUIRED_VALIDITY_MESSAGE}>Email <input required type="email" bind:this={emailInput} on:change={checkFormValidity}></label>
    <label data-validity-message={REQUIRED_VALIDITY_MESSAGE}>Password <input required type="password" bind:this={passwordInput} on:change={checkFormValidity}></label>
    <button disabled={!formValid} type="submit" on:click|preventDefault={() => { promise = login() }}>Login</button>
  </form>
{/if}

{#if error != null}
  <div>{error}</div>
{:else if user != null}
  <div>Hello {user.data?.firstName || ""}</div>
  <button on:click={logout}>Log out</button>
{/if}

{#await promise}
  <p>Loading...</p>
{/await}

<style>
  label {
    display: block;
  }

  input[type="email"], input[type="password"] {
    font-size: 16px;
  }

  label:after {
    content: attr(data-validity-message);
    padding-left: 5px;
  }
</style>