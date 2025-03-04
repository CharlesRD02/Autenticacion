Main Activity

package com.example.autenticacion

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.appcompat.app.AppCompatActivity
import androidx.biometric.BiometricManager
import androidx.biometric.BiometricPrompt
import androidx.biometric.BiometricPrompt.PromptInfo
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.core.content.ContextCompat

import com.example.autenticacion.ui.theme.AutenticacionTheme

class MainActivity : AppCompatActivity() {

    private var canAuthenticate = false
    private lateinit var promptInfo: BiometricPrompt.PromptInfo

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AutenticacionTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    Auth(::authenticate) // Pass the authenticate function
                }
            }
        }
        setupAuth()
    }

    private fun setupAuth() {
        val biometricManager = BiometricManager.from(this)
        if (biometricManager.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG or BiometricManager.Authenticators.DEVICE_CREDENTIAL) == BiometricManager.BIOMETRIC_SUCCESS) {
            canAuthenticate = true
            promptInfo = BiometricPrompt.PromptInfo.Builder()
                .setTitle("Autenticacion Biometrica")
                .setSubtitle("Autenticacion utilizando el sensor biometrico")
                .setAllowedAuthenticators(
                    BiometricManager.Authenticators.BIOMETRIC_STRONG or BiometricManager.Authenticators.DEVICE_CREDENTIAL
                )
                .build()
        }
    }

    private fun authenticate(auth: (auth: Boolean) -> Unit) {
        if (canAuthenticate) {
            BiometricPrompt(this, ContextCompat.getMainExecutor(this),
                object : BiometricPrompt.AuthenticationCallback() {
                    override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                        super.onAuthenticationSucceeded(result)
                        auth(true)
                    }

                    override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                        super.onAuthenticationError(errorCode, errString)
                        auth(false)
                    }
                    override fun onAuthenticationFailed() {
                        super.onAuthenticationFailed()
                        auth(false)
                    }
                }).authenticate(promptInfo)
        } else {
            auth(true) // If no biometric, consider authenticated.
        }
    }
}

@Composable
fun Auth(authenticate: (auth: (Boolean) -> Unit) -> Unit) { // Receive authenticate function

    var auth by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .background(if (auth) Color.Green else Color.Red)
            .fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            if (auth) "Estas Autenticado" else "Necesitas Autenticarte",
            fontSize = 22.sp,
            fontWeight = FontWeight.Bold
        )
        Spacer(Modifier.height(8.dp))
        Button(onClick = {
            if (auth) {
                auth = false
            } else {
                authenticate {
                    auth = it
                }
            }
        }) { Text(if (auth) "Cerrar" else "Autenticar") }
    }
}

@Preview(showSystemUi = true)
@Composable
fun DefaultPreview() {
    AutenticacionTheme {
        Auth(authenticate = {}) // Provide an empty lambda for preview
    }
}


