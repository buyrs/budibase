<script>
  import {
    Button,
    Heading,
    notifications,
    Layout,
    Body,
    Modal,
  } from "@budibase/bbui"
  import { goto } from "@roxi/routify"
  import { API } from "api"
  import { admin, auth } from "stores/portal"
  import ImportAppsModal from "./_components/ImportAppsModal.svelte"
  import Logo from "assets/bb-emblem.svg"
  import { onMount } from "svelte"
  import { FancyForm, FancyInput, ActionButton } from "@budibase/bbui"
  import { TestimonialPage } from "@budibase/frontend-core/src/components"
  import { passwordsMatch, handleError } from "../auth/_components/utils"

  let modal
  let form
  let errors = {}
  let formData = {}
  let submitted = false

  $: tenantId = $auth.tenantId
  $: cloud = $admin.cloud
  $: imported = $admin.importComplete

  async function save() {
    form.validate()
    if (Object.keys(errors).length > 0) {
      return
    }
    submitted = true
    try {
      let adminUser = { ...formData, tenantId }
      delete adminUser.confirmationPassword
      // Save the admin user
      await API.createAdminUser(adminUser)
      notifications.success("Admin user created")
      await admin.init()
      $goto("../portal")
    } catch (error) {
      submitted = false
      notifications.error("Failed to create admin user")
    }
  }

  onMount(async () => {
    if (!cloud) {
      try {
        await admin.checkImportComplete()
      } catch (error) {
        notifications.error("Error checking import status")
      }
    }
  })
</script>

<Modal bind:this={modal} padding={false} width="600px">
  <ImportAppsModal />
</Modal>

<TestimonialPage>
  <Layout gap="M" noPadding>
    <Layout justifyItems="center" noPadding>
      <img alt="logo" src={Logo} />
      <Heading size="M">Create an admin user</Heading>
      <Body>The admin user has access to everything in Budibase.</Body>
    </Layout>
    <Layout gap="S" noPadding>
      <FancyForm bind:this={form}>
        <FancyInput
          label="Email"
          value={formData.email}
          on:change={e => {
            formData = {
              ...formData,
              email: e.detail,
            }
          }}
          validate={() => {
            let fieldError = {
              email: !formData.email ? "Please enter a valid email" : undefined,
            }
            errors = handleError({ ...errors, ...fieldError })
          }}
          disabled={submitted}
          error={errors.email}
        />
        <FancyInput
          label="Password"
          value={formData.password}
          type="password"
          on:change={e => {
            formData = {
              ...formData,
              password: e.detail,
            }
          }}
          validate={() => {
            let fieldError = {}

            fieldError["password"] = !formData.password
              ? "Please enter a password"
              : undefined

            fieldError["confirmationPassword"] =
              !passwordsMatch(
                formData.password,
                formData.confirmationPassword
              ) && formData.confirmationPassword
                ? "Passwords must match"
                : undefined

            errors = handleError({ ...errors, ...fieldError })
          }}
          error={errors.password}
          disabled={submitted}
        />
        <FancyInput
          label="Repeat Password"
          value={formData.confirmationPassword}
          type="password"
          on:change={e => {
            formData = {
              ...formData,
              confirmationPassword: e.detail,
            }
          }}
          validate={() => {
            let fieldError = {
              confirmationPassword:
                !passwordsMatch(
                  formData.password,
                  formData.confirmationPassword
                ) && formData.password
                  ? "Passwords must match"
                  : undefined,
            }
            errors = handleError({ ...errors, ...fieldError })
          }}
          error={errors.confirmationPassword}
          disabled={submitted}
        />
      </FancyForm>
    </Layout>
    <Layout gap="XS" noPadding justifyItems="center">
      <Button
        cta
        disabled={Object.keys(errors).length > 0 || submitted}
        on:click={save}
      >
        Create super admin user
      </Button>
    </Layout>
    <Layout gap="XS" noPadding justifyItems="center">
      <div class="user-actions">
        {#if !cloud && !imported}
          <ActionButton
            quiet
            on:click={() => {
              modal.show()
            }}
          >
            Import from cloud
          </ActionButton>
        {/if}
      </div>
    </Layout>
  </Layout>
</TestimonialPage>

<style>
  img {
    width: 48px;
  }
</style>
