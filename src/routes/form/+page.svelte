<script lang="ts">
    let step = 1;

    let name = "";
    let address = "";
    let email = "";
    let teudatZehut = "";
    let creditCardDigits = "";

    type AgentType = "wifi" | "creditCard" | "insurance";
    let agentType: AgentType = "wifi";

    const descriptions = {
        wifi: ["Subscribe to New Plan", "Billing Issue", "Wifi Problem"],
        creditCard: ["Cancel Card", "Lost Card", "Fraud Alert", "Credit Limit Increase"],
        insurance: ["Claim Process", "Policy Renewal", "Premium Payment", "Coverage Details"]
    };

    const providers = {
        wifi: ["HOT", "Bezeq", "Partner", "Cellcom"],
        creditCard: ["MAX", "Cal", "Isracard"],
        insurance: ["Maccabi", "Clalit", "Meuhedet", "Leumit"]
    }

    let descriptionOptions = descriptions[agentType] || [];
    let providerOptions = providers[agentType] || [];

    let provider = providerOptions[0] || "";
    let description = descriptionOptions[0] || "";
    let userDescription = ""

    function updateAgentType() {
        descriptionOptions = descriptions[agentType];
        description = descriptionOptions[0] || "";

        providerOptions = providers[agentType];
        provider = providerOptions[0] || "";

    }

    function nextStep() {
        if (step === 1 && (!name || !address || !teudatZehut || !email)) {
        alert("Please fill in all required fields.");
        return;
        }
        step++;
    }

    function prevStep() {
        step--;
    }

    function handleSubmit(event: { preventDefault: () => void; }) {
      event.preventDefault();
      console.log("Form Submitted:", { 
        name,
        teudatZehut,
        email,
        address,
        creditCardDigits,
        agentType,
        provider,
        description,
        userDescription

    });
      // You can send this data to an API or process it further
    }

  </script>
  
  <style>
    form {
      max-width: 400px;
      margin: 100px auto;
      padding: 1.5rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      background: #f9f9f9;
    }
    label {
      font-weight: bold;
      margin-bottom: 4px;
      display: block;
    }
    select, textarea, input {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    button {
        background: #007bff;
        color: white;
        border: none;
        padding: 10px;
        cursor: pointer;
        width: 100%;
        border-radius: 4px;
        margin-top: 10px;
    }
    button:hover {
      background: #0056b3;
    }
  </style>
  
  <form on:submit={handleSubmit}>
    {#if step === 1}
        <label for="name">Full Name:</label>
        <input id="name" type="text" bind:value={name} required />

        <label for="teudat-zehut">Teudat Zehut:</label>
        <input id="teudat-zehut" type="text" bind:value={teudatZehut} required />

        <label for="email">Email:</label>
        <input id="email" type="text" bind:value={email} required />

        <label for="address">Address:</label>
        <input id="address" type="text" bind:value={address} required />

        <label for="credit-card-digits">Last 4 Digits of Credit Card:</label>
        <input id="credit-card-digits" type="text" bind:value={creditCardDigits} required />

        <button type="button" on:click={nextStep}>Next</button>
    {/if}

    {#if step === 2}
        <label for="agent-type">Task:</label>
        <select id="agent-type" bind:value={agentType} on:change={updateAgentType}>
            <option value="wifi">Wifi provider</option>
            <option value="creditCard">Credit card company</option>
            <option value="insurance">Health Insurance Company</option>
        </select>
    
        <label for="provider">Provider:</label>
        <select id="provider" bind:value={provider}>
        {#each providerOptions as option}
            <option value={option}>{option}</option>
        {/each}
        </select>

        <label for="description">Goal:</label>
        <select id="description" bind:value={description}>
        {#each descriptionOptions as option}
            <option value={option}>{option}</option>
        {/each}
        </select>

        <label for="user-description">Briefly describe your intentions and goals:</label>
        <textarea id="user-description" bind:value={userDescription} required></textarea>
    
    
        <button type="button" on:click={prevStep}>Back</button>
        <button type="submit">Kickstart Agent</button>
    {/if}
  </form>
  