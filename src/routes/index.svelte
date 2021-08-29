<script lang="ts">
  import axios from "axios"

  const REQUIRED_VALIDITY_MESSAGE = "Required"
  const baseUrl = "https://api-staging.holyart.io/v1/users/login"
  let user, error, formValid, promise

  const inputs = {
    "email": { "required": true, "value": "" },
    "password": { "required": true, "value": "" }
  }

  function checkFormValidity (ev) { 
    inputs[ev.target.getAttribute("type")].value = ev.target.value

    if (
      ev.target.parentElement.getAttribute("data-validity-message") === REQUIRED_VALIDITY_MESSAGE && ev.target.value !== "" ||
      ev.target.validity.valid
    )
      ev.target.parentElement.removeAttribute("data-validity-message")

    formValid = ev.target.form.checkValidity() 
  }

  function setValidityMessages (ev) {
    if (ev.target.validity.valid) ev.target.parentElement.removeAttribute("data-validity-message")
    else if (ev.target.hasAttribute("required") && ev.target.value === "") ev.target.parentElement.setAttribute("data-validity-message", REQUIRED_VALIDITY_MESSAGE)
    else ev.target.parentElement.setAttribute("data-validity-message", "Invalid " + ev.target.getAttribute("type"))
  }

  async function login () {
    return axios.post(baseUrl, {
      "domainCode": "it", "email": inputs.email.value, "password": inputs.password.value
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
    {#each Object.entries(inputs) as [type, { value, required }]}
      <label data-validity-message={required ? REQUIRED_VALIDITY_MESSAGE : null}>{type} <input {type} {required} on:input={checkFormValidity} on:blur={setValidityMessages}></label>
    {/each}
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
    text-transform: capitalize;
  }

  input[type="email"], input[type="password"] {
    font-size: 16px;
  }

  label:after {
    content: attr(data-validity-message);
    padding-left: 5px;
  }
</style>