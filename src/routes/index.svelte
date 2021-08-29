<script lang="ts">
  import axios from "axios"

  const REQUIRED_VALIDITY_MESSAGE = "Required"
  const baseUrl = "https://api-staging.holyart.io/v1/users/login"
  let user, error, formValid, promise, emailInput, passwordInput

  function checkFormValidity (ev) { 
    const inputType = ev.target.getAttribute("type")
    const validityMessageBox = ev.target.nextSibling

    if (ev.target.hasAttribute("required") && ev.target.value === "")
      validityMessageBox.setAttribute("data-validity-message", REQUIRED_VALIDITY_MESSAGE)
    else
      validityMessageBox.setAttribute("data-validity-message", "Invalid " + inputType)

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
    <div>
      <label>Email <input required type="email" bind:this={emailInput} on:input={checkFormValidity}><span data-validity-message={REQUIRED_VALIDITY_MESSAGE}></span></label>
    </div>
    <div>
      <label>Password <input required type="password" bind:this={passwordInput} on:input={checkFormValidity}><span data-validity-message={REQUIRED_VALIDITY_MESSAGE}></span></label>
    </div>
    <button disabled={!formValid} type="submit" on:click|preventDefault={() => { promise = login() }}>Login</button>
  </form>
{#await promise}
  <p>Loading...</p>
{/await}
{/if}


{#if error != null}
  <div>{error}</div>
{:else if user != null}
  <div>Hello {user.data?.firstName || ""}</div>
  <button on:click={logout}>Log out</button>
{/if}

<style>
  input[type="email"],
  input[type="password"] {
    font-size: 16px;
  }

  input:invalid+span:after {
    content: attr(data-validity-message);
    padding-left: 5px;
  }
</style>