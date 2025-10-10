import React, { useState, useEffect } from 'react';
import { Mail, Lock, Eye, EyeOff, User, RotateCw, CheckCircle, ArrowLeft, Key, Zap, MessageSquare, Lightbulb } from 'lucide-react';

// Assume Tailwind CSS is configured, so we use its utility classes directly.

// Helper component for styled input fields
const InputField = ({ label, type, value, onChange, placeholder, icon: Icon, required = true, isPassword = false, showPassword, togglePasswordVisibility }) => (
  <div className="space-y-1">
    <label htmlFor={type} className="block text-sm font-medium text-gray-700">{label}</label>
    <div className="relative flex items-center">
      <div className="absolute left-0 pl-3 flex items-center pointer-events-none">
        <Icon className="h-5 w-5 text-gray-400" />
      </div>
      <input
        type={isPassword ? (showPassword ? 'text' : 'password') : type}
        id={type}
        className="block w-full pl-10 pr-12 py-3 border border-gray-300 rounded-lg bg-gray-50 text-gray-900 placeholder-gray-500 focus:ring-blue-500 focus:border-blue-500 transition duration-150"
        placeholder={placeholder}
        value={value}
        onChange={onChange}
        required={required}
      />
      {isPassword && (
        <span 
          className="absolute right-0 pr-3 flex items-center cursor-pointer text-gray-500 hover:text-gray-700"
          onClick={togglePasswordVisibility}
        >
          {showPassword ? <EyeOff className="h-5 w-5" /> : <Eye className="h-5 w-5" />}
        </span>
      )}
    </div>
  </div>
);

// Main Application Component
const LoginApp = () => {
  const apiKey = "";
  const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
  
  // State for view management: 'login', 'forgotPassword', 'resetPassword', 'success'
  const [currentView, setCurrentView] = useState('login'); 
  
  // State for form inputs
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [otp, setOtp] = useState('');
  const [newPassword, setNewPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');

  // LLM Feature States (Password Analyzer - in Reset Form)
  const [passwordAnalysisResult, setPasswordAnalysisResult] = useState(null);
  const [isAnalysisLoading, setIsAnalysisLoading] = useState(false);
  
  // LLM Feature States (Security Tips - in Forgot Password Form)
  const [securityTips, setSecurityTips] = useState(null);
  const [isTipsLoading, setIsTipsLoading] = useState(false);

  // UI state
  const [showPassword, setShowPassword] = useState(false); 
  const [showNewPassword, setShowNewPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState('');

  const togglePasswordVisibility = (setter) => {
    setter(prev => !prev);
  };
  
  // --- Gemini API Function: Analyze Password Strength (Feature 1) ---

  const handlePasswordAnalysis = async (password) => {
      if (!password) {
          setPasswordAnalysisResult({ reason: 'Please enter a password to analyze.', strength: 'Weak' });
          return;
      }

      setIsAnalysisLoading(true);
      setPasswordAnalysisResult(null);

      const systemPrompt = "You are a specialized security analyst for Standard Chartered Bank. Analyze the provided password. Respond in JSON format only with the following structure: { \"strength\": \"Weak\" | \"Medium\" | \"Strong\", \"reason\": \"A concise explanation of why, focusing on complexity, length, and variety of characters. Be encouraging.\", \"suggestion\": \"A highly secure, complex password suggestion that is 16 characters or longer. Do not use dictionary words.\" }";

      const userQuery = `Analyze the following password: "${password}"`;

      const payload = {
          contents: [{ parts: [{ text: userQuery }] }],
          systemInstruction: { parts: [{ text: systemPrompt }] },
          generationConfig: {
              responseMimeType: "application/json",
              responseSchema: {
                  type: "OBJECT",
                  properties: {
                      "strength": { "type": "STRING", "description": "Weak, Medium, or Strong" },
                      "reason": { "type": "STRING", "description": "Why the password has this strength." },
                      "suggestion": { "type": "STRING", "description": "A very strong, non-dictionary password suggestion." }
                  },
                  propertyOrdering: ["strength", "reason", "suggestion"]
              }
          }
      };

      try {
          let response;
          let result;
          let parsedJson = null;
          let attempts = 0;
          const maxAttempts = 3;

          while (attempts < maxAttempts) {
              try {
                  response = await fetch(apiUrl, {
                      method: 'POST',
                      headers: { 'Content-Type': 'application/json' },
                      body: JSON.stringify(payload)
                  });

                  if (response.ok) {
                      result = await response.json();
                      const jsonText = result?.candidates?.[0]?.content?.parts?.[0]?.text;
                      if (jsonText) {
                          parsedJson = JSON.parse(jsonText);
                          break;
                      }
                  }
              } catch (e) {
                  // If it's a network error or JSON parsing error, retry (with backoff)
              }

              attempts++;
              if (attempts < maxAttempts) {
                  const delay = Math.pow(2, attempts) * 1000;
                  await new Promise(resolve => setTimeout(resolve, delay));
              }
          }

          if (parsedJson) {
              // Ensure we handle case sensitivity/typos from LLM
              const strength = parsedJson.strength?.toLowerCase();
              const isStrong = strength === 'strong';

              setPasswordAnalysisResult({ 
                ...parsedJson, 
                isStrong,
                // Map strength to Tailwind color classes
                color: isStrong ? 'green' : (strength === 'medium' ? 'yellow' : 'red')
              });
          } else {
              throw new Error("Failed to get a structured response from the API.");
          }

      } catch (error) {
          console.error("Gemini API error:", error);
          setPasswordAnalysisResult({ 
            strength: 'Error', 
            reason: 'Could not connect to the analysis service. Please try again.', 
            isStrong: false,
            color: 'red' 
          });
      } finally {
          setIsAnalysisLoading(false);
      }
  };

  // --- Gemini API Function: Generate Security Tips (Feature 2) ---

  const handleSecurityTipsGeneration = async () => {
    setIsTipsLoading(true);
    setSecurityTips(null);

    const systemPrompt = "You are a friendly, professional digital security specialist for a bank. Your task is to generate exactly 3 concise, actionable, and vital security tips specifically related to email and password hygiene for a customer who is about to reset their password. Respond only in JSON format.";

    const userQuery = "Generate 3 crucial security tips for password and email management for a bank customer.";

    const payload = {
        contents: [{ parts: [{ text: userQuery }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
        generationConfig: {
            responseMimeType: "application/json",
            responseSchema: {
                type: "ARRAY",
                items: {
                    type: "OBJECT",
                    properties: {
                        "tip": { "type": "STRING", "description": "A concise and actionable security tip." },
                    },
                    propertyOrdering: ["tip"]
                }
            }
        }
    };
    
    try {
        let response;
        let result;
        let parsedJson = null;
        let attempts = 0;
        const maxAttempts = 3;

        while (attempts < maxAttempts) {
            try {
                response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (response.ok) {
                    result = await response.json();
                    const jsonText = result?.candidates?.[0]?.content?.parts?.[0]?.text;
                    if (jsonText) {
                        parsedJson = JSON.parse(jsonText);
                        break;
                    }
                }
            } catch (e) {
                // If it's a network error or JSON parsing error, retry (with backoff)
            }

            attempts++;
            if (attempts < maxAttempts) {
                const delay = Math.pow(2, attempts) * 1000;
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }

        if (parsedJson && Array.isArray(parsedJson)) {
            setSecurityTips(parsedJson);
        } else {
            throw new Error("Failed to get structured security tips response.");
        }
    } catch (error) {
        console.error("Gemini API error for tips:", error);
        // Fallback tips
        setSecurityTips([
            { tip: "Use unique passwords for all major accounts." },
            { tip: "Enable two-factor authentication (2FA) wherever possible." },
            { tip: "Never click suspicious links in emails, even if they look legitimate." }
        ]);
    } finally {
        setIsTipsLoading(false);
    }
  };


  // --- Effect to trigger tips generation when entering the Forgot Password view ---
  useEffect(() => {
    if (currentView === 'forgotPassword') {
        handleSecurityTipsGeneration();
    }
    // Clear analysis state when moving away from reset page
    if (currentView !== 'resetPassword') {
        setPasswordAnalysisResult(null);
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [currentView]);


  // --- Core Logic Functions (Unchanged) ---

  const handleLoginSubmit = (event) => {
    event.preventDefault();
    setMessage('');
    setLoading(true);

    if (email === 'demo@sc.com' && password === 'password') {
      setTimeout(() => {
        setLoading(false);
        setMessage('Login successful! Welcome back.');
      }, 1500);
    } else {
      setTimeout(() => {
        setLoading(false);
        setMessage('Login failed. Invalid email or password.');
      }, 1500);
    }
  };

  const handleForgotPasswordSubmit = (event) => {
    event.preventDefault();
    setMessage('');
    setLoading(true);
    
    // Simulate API call to send OTP/Reset Link
    setTimeout(() => {
      setLoading(false);
      if (email) {
        setMessage(`OTP simulation sent to ${email}. Proceed to the reset page.`);
        setCurrentView('resetPassword');
      } else {
        setMessage('Please provide a valid email address.');
      }
    }, 2000);
  };

  const handleResetPasswordSubmit = (event) => {
    event.preventDefault();
    setMessage('');
    setLoading(true);

    if (newPassword !== confirmPassword) {
      setLoading(false);
      setMessage('New password and confirmation password must match.');
      return;
    }
    
    // Simulate API call to verify OTP and set new password
    setTimeout(() => {
      setLoading(false);
      if (otp === '123456') { // Simple hardcoded OTP for demo
        setMessage('Password successfully reset! Redirecting to login...');
        setCurrentView('success');
        // Clear fields and analysis on success
        setNewPassword('');
        setConfirmPassword('');
        setOtp('');
        setPasswordAnalysisResult(null);
        setTimeout(() => setCurrentView('login'), 3000);
      } else {
        setMessage('OTP verification failed. Please try again.');
      }
    }, 2000);
  };

  // --- View Rendering Functions ---
  
  const renderLoginForm = () => (
    <form onSubmit={handleLoginSubmit} className="space-y-5">
      <InputField
        label="Email Address"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
        icon={User}
      />
      <InputField
        label="Password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Enter your password"
        icon={Lock}
        isPassword={true}
        showPassword={showPassword}
        togglePasswordVisibility={() => togglePasswordVisibility(setShowPassword)}
      />
      
      {/* Forgot Password Link */}
      <div className="flex justify-end text-sm">
        <button 
          type="button" 
          onClick={() => setCurrentView('forgotPassword')}
          className="font-medium text-blue-600 hover:text-blue-500 transition duration-150"
        >
          Forgot Password?
        </button>
      </div>
      
      {/* Log In Button */}
      <div className="pt-2">
        <button 
          type="submit" 
          className="w-full flex justify-center items-center py-3 px-4 border border-transparent text-lg font-bold rounded-lg shadow-md text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-200 disabled:opacity-50"
          disabled={loading}
        >
          {loading ? <RotateCw className="animate-spin h-5 w-5 mr-3" /> : 'Log In'}
        </button>
      </div>

      {/* Sign Up Link */}
      <div className="text-center pt-4">
        <p className="text-sm text-gray-600">
          Don't have an account? 
          <a href="/signup" className="ml-1 font-medium text-blue-600 hover:text-blue-500">
            Sign Up
          </a>
        </p>
      </div>
    </form>
  );

  const renderForgotPasswordForm = () => (
    <form onSubmit={handleForgotPasswordSubmit} className="space-y-5">
      <h3 className="text-xl font-semibold text-gray-800 pt-2 mb-4 text-center">Reset Password</h3>
      <p className="text-sm text-gray-600 text-center">
        Enter your email address to receive a one-time code to reset your password.
      </p>

      <InputField
        label="Email Address"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
        icon={Mail}
      />
      
      {/* Security Tips Generator (Feature 2) */}
      <div className="pt-6">
        <h4 className="text-md font-semibold text-gray-800 flex items-center mb-2">
            <Lightbulb className="w-5 h-5 mr-2 text-yellow-500 fill-yellow-100" />
            Security Best Practices ✨
        </h4>
        {isTipsLoading && (
            <div className="text-center p-4 text-sm text-gray-500">
                <RotateCw className="animate-spin h-5 w-5 mx-auto mb-2" />
                Generating vital security tips...
            </div>
        )}
        {securityTips && (
            <ul className="space-y-2 text-sm text-gray-700 bg-yellow-50 p-4 rounded-lg border border-yellow-200">
                {securityTips.map((tipObj, index) => (
                    <li key={index} className="flex items-start">
                        <span className="text-yellow-500 font-bold text-lg leading-none mr-2 mt-0.5">•</span>
                        <span>{tipObj.tip}</span>
                    </li>
                ))}
            </ul>
        )}
      </div>

      
      {/* Send Code Button */}
      <div className="pt-4">
        <button 
          type="submit" 
          className="w-full flex justify-center items-center py-3 px-4 border border-transparent text-lg font-bold rounded-lg shadow-md text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition duration-200 disabled:opacity-50"
          disabled={loading}
        >
          {loading ? <RotateCw className="animate-spin h-5 w-5 mr-3" /> : 'Send Reset Code'}
        </button>
      </div>

      {/* Back to Login */}
      <div className="text-center pt-2">
        <button 
          type="button" 
          onClick={() => setCurrentView('login')}
          className="text-sm font-medium text-gray-600 hover:text-gray-800 flex items-center mx-auto transition duration-150"
        >
          <ArrowLeft className="w-4 h-4 mr-1" />
          Back to Login
        </button>
      </div>
    </form>
  );

  const renderResetPasswordForm = () => (
    <form onSubmit={handleResetPasswordSubmit} className="space-y-5">
      <h3 className="text-xl font-semibold text-gray-800 pt-2 mb-4 text-center">Verify & Set New Password</h3>
      <p className="text-sm text-gray-600 text-center">
        A code has been sent to **{email}**. Enter the code and your new password below.
      </p>

      <InputField
        label="One-Time Password (OTP)"
        type="text"
        value={otp}
        onChange={(e) => setOtp(e.target.value)}
        placeholder="Enter 6-digit code (Demo: 123456)"
        icon={Key}
        required
      />

      {/* New Password Input */}
      <InputField
        label="New Password"
        type="password"
        value={newPassword}
        onChange={(e) => {setNewPassword(e.target.value); setPasswordAnalysisResult(null);}}
        placeholder="Create a new password"
        icon={Lock}
        isPassword={true}
        showPassword={showNewPassword}
        togglePasswordVisibility={() => togglePasswordVisibility(setShowNewPassword)}
      />

      {/* Gemini Password Analysis Feature (Feature 1) */}
      <div className="pt-1">
        <button
            type="button"
            onClick={() => handlePasswordAnalysis(newPassword)}
            disabled={isAnalysisLoading}
            className="w-full flex justify-center items-center py-2 px-4 border border-blue-200 text-sm font-bold rounded-lg shadow-sm text-blue-600 bg-blue-50 hover:bg-blue-100 transition duration-150 disabled:opacity-50"
        >
            {isAnalysisLoading ? (
                <RotateCw className="animate-spin h-4 w-4 mr-2" />
            ) : (
                <Zap className="h-4 w-4 mr-2 fill-blue-500 text-blue-500" />
            )}
            {isAnalysisLoading ? 'Analyzing...' : 'Analyze Password Strength ✨'}
        </button>
      </div>
      
      {/* Password Analysis Result Display */}
      {passwordAnalysisResult && (
          <div className={`p-4 rounded-lg border-l-4 ${
              passwordAnalysisResult.color === 'green' ? 'bg-green-50 border-green-500 text-green-700' :
              passwordAnalysisResult.color === 'yellow' ? 'bg-yellow-50 border-yellow-500 text-yellow-700' :
              'bg-red-50 border-red-500 text-red-700'
          } shadow-sm transition duration-300`}>
              <div className="flex items-center mb-1">
                  <MessageSquare className="w-4 h-4 mr-2" />
                  <span className="font-semibold text-sm">Strength: {passwordAnalysisResult.strength}</span>
              </div>
              <p className="text-xs italic mb-2">{passwordAnalysisResult.reason}</p>

              {passwordAnalysisResult.suggestion && (
                <div className="mt-2 pt-2 border-t border-gray-200">
                    <p className="text-xs font-semibold mb-1">Highly Secure Suggestion:</p>
                    <code 
                        className="text-xs bg-white p-1 rounded border border-dashed border-gray-300 break-all cursor-pointer hover:bg-gray-100 transition"
                        onClick={() => setNewPassword(passwordAnalysisResult.suggestion)}
                        title="Click to use this suggestion"
                    >
                        {passwordAnalysisResult.suggestion}
                    </code>
                </div>
              )}
          </div>
      )}


      <InputField
        label="Confirm New Password"
        type="password"
        value={confirmPassword}
        onChange={(e) => setConfirmPassword(e.target.value)}
        placeholder="Confirm your new password"
        icon={Lock}
        isPassword={true}
        showPassword={showConfirmPassword}
        togglePasswordVisibility={() => togglePasswordVisibility(setShowConfirmPassword)}
      />
      
      {/* Reset Button */}
      <div className="pt-4">
        <button 
          type="submit" 
          className="w-full flex justify-center items-center py-3 px-4 border border-transparent text-lg font-bold rounded-lg shadow-md text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-200 disabled:opacity-50"
          disabled={loading}
        >
          {loading ? <RotateCw className="animate-spin h-5 w-5 mr-3" /> : 'Reset Password'}
        </button>
      </div>

      {/* Back to Forgot Password */}
      <div className="text-center pt-2">
        <button 
          type="button" 
          onClick={() => setCurrentView('forgotPassword')}
          className="text-sm font-medium text-gray-600 hover:text-gray-800 flex items-center mx-auto transition duration-150"
        >
          <ArrowLeft className="w-4 h-4 mr-1" />
          Back to Email Entry
        </button>
      </div>
    </form>
  );

  const renderSuccessMessage = () => (
    <div className="space-y-5 text-center py-8">
      <CheckCircle className="w-16 h-16 text-green-500 mx-auto" />
      <h3 className="text-2xl font-bold text-gray-800">Success!</h3>
      <p className="text-lg text-gray-600">Your password has been reset.</p>
      <p className="text-sm text-gray-500">Redirecting you to the login screen...</p>
    </div>
  );

  // --- Main Render Structure ---
  return (
    <div className="min-h-screen flex items-center justify-center p-4 sm:p-6"
      style={{
        backgroundImage: 'linear-gradient(135deg, #F3FCF8 0%, #DFF0F3 50%, #E6F3EE 100%)',
      }}
    >
      <div className="w-full max-w-md bg-white p-6 sm:p-8 rounded-2xl shadow-2xl border border-gray-100">
        
        {/* Logo and Branding Section */}
        <div className="text-center mb-6">
          {/* Using a placeholder image URL for the logo */}
          <img 
            src="https://placehold.co/60x60/007bff/ffffff?text=SC" 
            alt="Standard Chartered Logo" 
            className="w-16 h-16 object-contain mx-auto mb-3 rounded-full shadow-md"
          />
          <h2 className="text-2xl font-extrabold text-gray-800">Standard Chartered Bank</h2>
          <p className="text-md font-medium text-gray-500">
            {currentView === 'login' ? 'Sign In to Your Account' : 
             currentView === 'forgotPassword' ? 'Initiate Password Reset' : 
             currentView === 'resetPassword' ? 'Complete Password Reset' : 
             'Process Complete'}
          </p>
        </div>
        
        {/* Dynamic Message/Alert Box */}
        {message && (
          <div className={`p-3 mb-4 rounded-lg text-sm font-medium ${message.includes('success') ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}`}>
            {message}
          </div>
        )}

        {/* Dynamic View Switcher */}
        {currentView === 'login' && renderLoginForm()}
        {currentView === 'forgotPassword' && renderForgotPasswordForm()}
        {currentView === 'resetPassword' && renderResetPasswordForm()}
        {currentView === 'success' && renderSuccessMessage()}

      </div>
    </div>
  );
};

export default LoginApp;
