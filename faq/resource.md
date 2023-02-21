---
description: >-
  AdminJS does not currently provide an out-of-the-box solution for password
  recovery. You will need to implement your own way of resetting the password.
---

# Forgot Password

In order to add a custom Forgot Password link to Admin's login page you will have to, first, override the login within your start function and provide another custom component that will serve as a login page.

```typescript
admin.overrideLogin({ component: Login })
```

The next step is to create a login page. Here is the default page with the forgot password hyperlink pointing to a  '/forgot-password' address.

{% code title="login.tsx" %}
```typescript
import React from 'react'
import styled, { createGlobalStyle } from 'styled-components'
import { useSelector } from 'react-redux'
import {
  Box,
  H5,
  H2,
  Label,
  Illustration,
  Input,
  FormGroup,
  Button,
  Text,
  MessageBox,
  MadeWithLove,
  themeGet,
} from '@adminjs/design-system'
import { useTranslation, ReduxState } from 'adminjs'

const GlobalStyle = createGlobalStyle`
  html, body, #app {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
  }
`

const Wrapper = styled(Box)`
  align-items: center;
  justify-content: center;
  flex-direction: column;
  height: 100%;
`

const StyledLogo = styled.img`
  max-width: 200px;
  margin: ${themeGet('space', 'md')} 0;
`

export type LoginProps = {
  message?: string
  action: string
}

export const Login: React.FC<LoginProps> = (props) => {
  const { action, message } = props
  const { translateLabel, translateButton, translateProperty, translateMessage } = useTranslation()
  const branding = useSelector((state: ReduxState) => state.branding)

  return (
    <>
      <GlobalStyle />
      <Wrapper flex variant="grey">
        <Box bg="white" height="440px" flex boxShadow="login" width={[1, 2 / 3, 'auto']}>
          <Box
            bg="primary100"
            color="white"
            p="x3"
            width="380px"
            flexGrow={0}
            display={['none', 'none', 'block']}
            position="relative"
          >
            <H2 fontWeight="lighter">{translateLabel('loginWelcome')}</H2>
            <Text fontWeight="lighter" mt="default">
              {translateMessage('loginWelcome')}
            </Text>
            <Text textAlign="center" p="xxl">
              <Box display="inline" mr="default">
                <Illustration variant="Planet" width={82} height={91} />
              </Box>
              <Box display="inline">
                <Illustration variant="Astronaut" width={82} height={91} />
              </Box>
              <Box display="inline" position="relative" top="-20px">
                <Illustration variant="FlagInCog" width={82} height={91} />
              </Box>
            </Text>
          </Box>
          <Box as="form" action={action} method="POST" p="x3" flexGrow={1} width={['100%', '100%', '480px']}>
            <H5 marginBottom="xxl">
              {branding.logo ? <StyledLogo src={branding.logo} alt={branding.companyName} /> : branding.companyName}
            </H5>
            {message && (
              <MessageBox
                my="lg"
                message={message.split(' ').length > 1 ? message : translateMessage(message)}
                variant="danger"
              />
            )}
            <FormGroup>
              <Label required>{translateProperty('email')}</Label>
              <Input name="email" placeholder={translateProperty('email')} />
            </FormGroup>
            <FormGroup>
              <Label required>{translateProperty('password')}</Label>
              <Input
                type="password"
                name="password"
                placeholder={translateProperty('password')}
                autoComplete="new-password"
              />
            </FormGroup>
            <Text mt="xl" textAlign="center">
              <Button variant="primary">{translateButton('login')}</Button>
            </Text>
            <Text mt="lg" textAlign="center">
              {translateMessage('forgotPasswordQuestion')}{' '}
              <a href="/forgot-password">{translateMessage('forgotPassword')}</a>
            </Text>
          </Box>
        </Box>
        {branding.withMadeWithLove ? (
          <Box mt="xxl">
            <MadeWithLove />
          </Box>
        ) : null}
      </Wrapper>
    </>
  )
}

export default Login

```
{% endcode %}

Keep in mind that to keep the custom login page responsive, you will need to provide locale translation for the custom items we have added.

```typescript
const localeEn = {
  language: 'en',
  translations: {
    messages: {
      forgotPasswordQuestion: 'Trouble logging in?',
      forgotPassword: 'Forgot password'
    }
  }
}

export default localeEn

```

Now, depending on your authorization implementation, you will have to handle the request from the 'Forgot password' hyperlink pointing to the '/forgot-password' address.
