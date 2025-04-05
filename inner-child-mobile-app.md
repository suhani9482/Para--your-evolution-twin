# PARA Evolution: Inner Child Pet Mobile Application

## Overview
This document outlines the technical framework for converting the Inner Child Pet feature into a functional mobile application. The app represents the user's inner child as a pet kitten that grows alongside the user during their 41-day emotional evolution journey.

## Technical Stack

### Frontend
- **Framework**: React Native
- **UI Kit**: Native Base or UI Kitten for cross-platform UI components
- **State Management**: Redux with Redux Persist for local storage
- **Animations**: React Native Animated API + Lottie for fluid animations

### Backend
- **Server**: Firebase or AWS Amplify
- **Authentication**: Firebase Auth or AWS Cognito
- **Database**: Firebase Firestore/Realtime DB or AWS DynamoDB
- **Storage**: Firebase Storage or AWS S3 (for user-uploaded content)
- **Notifications**: Firebase Cloud Messaging for daily reminders

### Additional Dependencies
- React Native Gesture Handler (for interactive pet animations)
- React Navigation (for app navigation)
- Async Storage (for local data persistence)
- Expo Notifications (for local and push notifications)
- Expo Haptics (for tactile feedback when interacting with pet)

## App Architecture

### Core Features

1. **Pet Growth System**
   - 5 growth stages representing inner child development
   - Evolution triggers based on emotional expression consistency
   - Visual transformations with celebration animations

2. **Emotion Tracking**
   - Daily emotion logging with intensity metrics
   - Pattern recognition to identify recurring emotional states
   - Historical emotion data visualization

3. **Reflection Journal**
   - Voice-to-text option for easier expression
   - Guided reflection prompts based on detected emotions
   - Private, encrypted storage of all journal entries

4. **Nurturing Mechanics**
   - Feeding the pet with emotional expression (primary nurturing action)
   - "Pet comfort" interactions using haptic feedback and animations
   - Rewards for consistency and emotional honesty

5. **Notification System**
   - Gentle reminders when pet "needs attention" (no emotional expression for 24+ hours)
   - Celebration notifications for growth milestones
   - Morning/evening check-in nudges customized to user preferences

6. **Progress Visualization**
   - 41-day timeline with clear growth tracking
   - Need meters for Love, Expression, and Comfort 
   - Achievement badges for emotional growth milestones

7. **Offline Functionality**
   - Full app usage without internet connection
   - Background data sync when connection restored
   - Local storage of all user data with encryption

## Implementation Plan

### 1. Core Pet System

```javascript
// Pet State Management
const initialPetState = {
  name: "Whispers",
  stage: 1,
  currentDay: 1,
  totalDays: 41,
  needLevels: {
    love: 20,
    expression: 10,
    comfort: 15
  },
  emotion: "neutral",
  milestones: [
    { day: 1, completed: false, badge: "🌱", title: "Beginning the Journey" },
    { day: 10, completed: false, badge: "🌿", title: "First Sprouts" },
    { day: 20, completed: false, badge: "🌺", title: "Emotional Blossoming" },
    { day: 30, completed: false, badge: "🦁", title: "Inner Strength" },
    { day: 41, completed: false, badge: "✨", title: "True Evolution" }
  ],
  reflectionStreak: 0,
  emotionHistory: []
};

// Growth Algorithm
function calculatePetGrowth(currentState, newReflection) {
  // Clone state to avoid mutations
  const updatedState = {...currentState};
  
  // Update needs based on reflection
  if (newReflection.text.length > 50) {
    updatedState.needLevels.expression += 15;
  } else {
    updatedState.needLevels.expression += 5;
  }
  
  // Emotional content analysis increases comfort
  if (newReflection.emotions.length > 0) {
    updatedState.needLevels.comfort += 10;
    
    // Check for vulnerable emotions (increases love)
    const vulnerableEmotions = ['sad', 'afraid', 'anxious', 'lonely'];
    if (newReflection.emotions.some(e => vulnerableEmotions.includes(e))) {
      updatedState.needLevels.love += 15;
    } else {
      updatedState.needLevels.love += 5;
    }
  }
  
  // Cap all needs at 100
  Object.keys(updatedState.needLevels).forEach(need => {
    updatedState.needLevels[need] = Math.min(updatedState.needLevels[need], 100);
  });
  
  // Check for stage advancement
  const totalNeedLevel = Object.values(updatedState.needLevels).reduce((sum, val) => sum + val, 0);
  const averageNeed = totalNeedLevel / 3;
  
  if (averageNeed > 75 && updatedState.reflectionStreak >= 5 && updatedState.stage < 5) {
    updatedState.stage += 1;
    // Reset needs after advancement to create new growth cycle
    Object.keys(updatedState.needLevels).forEach(need => {
      updatedState.needLevels[need] = Math.max(updatedState.needLevels[need] - 30, 20);
    });
  }
  
  // Update reflection streak
  updatedState.reflectionStreak += 1;
  
  // Add to emotion history
  updatedState.emotionHistory.push({
    day: updatedState.currentDay,
    emotions: newReflection.emotions,
    text: newReflection.text,
    date: new Date().toISOString()
  });
  
  return updatedState;
}
```

### 2. User Interface Components

#### Main Pet Screen
The main interaction screen with the pet animation, need meters, and quick emotion logging:

```jsx
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, Animated, TouchableOpacity, TextInput } from 'react-native';
import { useSelector, useDispatch } from 'react-redux';
import * as Haptics from 'expo-haptics';
import LottieView from 'lottie-react-native';

export const PetScreen = () => {
  const petState = useSelector(state => state.pet);
  const dispatch = useDispatch();
  const [reflectionText, setReflectionText] = useState('');
  const [selectedEmotions, setSelectedEmotions] = useState([]);
  const growthAnimation = useRef(null);
  
  // Animated values
  const loveFillAnim = useRef(new Animated.Value(petState.needLevels.love)).current;
  const expressionFillAnim = useRef(new Animated.Value(petState.needLevels.expression)).current;
  const comfortFillAnim = useRef(new Animated.Value(petState.needLevels.comfort)).current;
  
  // Update animated values when pet state changes
  useEffect(() => {
    Animated.parallel([
      Animated.timing(loveFillAnim, {
        toValue: petState.needLevels.love,
        duration: 1000,
        useNativeDriver: false
      }),
      Animated.timing(expressionFillAnim, {
        toValue: petState.needLevels.expression,
        duration: 1000,
        useNativeDriver: false
      }),
      Animated.timing(comfortFillAnim, {
        toValue: petState.needLevels.comfort,
        duration: 1000,
        useNativeDriver: false
      })
    ]).start();
  }, [petState.needLevels]);
  
  const petStageAnimations = {
    1: require('../assets/animations/kitten_stage1.json'),
    2: require('../assets/animations/kitten_stage2.json'),
    3: require('../assets/animations/kitten_stage3.json'),
    4: require('../assets/animations/kitten_stage4.json'),
    5: require('../assets/animations/kitten_stage5.json'),
  };
  
  const emotions = [
    { id: 'anxious', label: 'Anxious', icon: '😰' },
    { id: 'sad', label: 'Sad', icon: '😢' },
    { id: 'angry', label: 'Angry', icon: '😠' },
    { id: 'happy', label: 'Happy', icon: '😊' },
    { id: 'peaceful', label: 'Peaceful', icon: '😌' },
    { id: 'lonely', label: 'Lonely', icon: '🥺' },
    { id: 'grateful', label: 'Grateful', icon: '🙏' },
    { id: 'proud', label: 'Proud', icon: '🦋' },
  ];
  
  const toggleEmotion = (emotionId) => {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
    
    if (selectedEmotions.includes(emotionId)) {
      setSelectedEmotions(selectedEmotions.filter(id => id !== emotionId));
    } else {
      setSelectedEmotions([...selectedEmotions, emotionId]);
    }
  };
  
  const submitReflection = () => {
    if (reflectionText.trim() === '' || selectedEmotions.length === 0) return;
    
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    
    dispatch({
      type: 'ADD_REFLECTION',
      payload: {
        text: reflectionText,
        emotions: selectedEmotions
      }
    });
    
    // Reset form
    setReflectionText('');
    setSelectedEmotions([]);
    
    // Play growth animation if applicable
    if (growthAnimation.current) {
      growthAnimation.current.play();
    }
  };
  
  return (
    <View style={styles.container}>
      <View style={styles.petContainer}>
        <LottieView
          ref={growthAnimation}
          source={petStageAnimations[petState.stage]}
          autoPlay
          loop
          style={styles.petAnimation}
        />
        
        <View style={styles.needsContainer}>
          <View style={styles.needItem}>
            <Text style={styles.needLabel}>Love</Text>
            <View style={styles.needBar}>
              <Animated.View 
                style={[
                  styles.needFill, 
                  styles.loveFill,
                  { width: loveFillAnim.interpolate({
                    inputRange: [0, 100],
                    outputRange: ['0%', '100%']
                  })}
                ]} 
              />
            </View>
          </View>
          
          <View style={styles.needItem}>
            <Text style={styles.needLabel}>Expression</Text>
            <View style={styles.needBar}>
              <Animated.View 
                style={[
                  styles.needFill, 
                  styles.expressionFill,
                  { width: expressionFillAnim.interpolate({
                    inputRange: [0, 100],
                    outputRange: ['0%', '100%']
                  })}
                ]} 
              />
            </View>
          </View>
          
          <View style={styles.needItem}>
            <Text style={styles.needLabel}>Comfort</Text>
            <View style={styles.needBar}>
              <Animated.View 
                style={[
                  styles.needFill, 
                  styles.comfortFill,
                  { width: comfortFillAnim.interpolate({
                    inputRange: [0, 100],
                    outputRange: ['0%', '100%']
                  })}
                ]} 
              />
            </View>
          </View>
        </View>
      </View>
      
      <Text style={styles.reflectionTitle}>How are you feeling today?</Text>
      
      <View style={styles.emotionsContainer}>
        {emotions.map(emotion => (
          <TouchableOpacity
            key={emotion.id}
            style={[
              styles.emotionChip,
              selectedEmotions.includes(emotion.id) && styles.selectedEmotionChip
            ]}
            onPress={() => toggleEmotion(emotion.id)}
          >
            <Text style={styles.emotionIcon}>{emotion.icon}</Text>
            <Text style={[
              styles.emotionLabel,
              selectedEmotions.includes(emotion.id) && styles.selectedEmotionLabel
            ]}>
              {emotion.label}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      
      <TextInput
        style={styles.reflectionInput}
        placeholder="Express what's on your mind... your inner child is listening"
        multiline
        value={reflectionText}
        onChangeText={setReflectionText}
      />
      
      <TouchableOpacity 
        style={[
          styles.submitButton,
          (reflectionText.trim() === '' || selectedEmotions.length === 0) && styles.disabledButton
        ]}
        onPress={submitReflection}
        disabled={reflectionText.trim() === '' || selectedEmotions.length === 0}
      >
        <Text style={styles.submitButtonText}>Feed Your Inner Child</Text>
      </TouchableOpacity>
      
      <Text style={styles.dayCounter}>
        Day {petState.currentDay} of {petState.totalDays}
      </Text>
    </View>
  );
};
```

### 3. Database Schema

```javascript
// Firestore Database Schema

// Users Collection
users: {
  userId: {
    email: string,
    displayName: string,
    createdAt: timestamp,
    journeyStartDate: timestamp,
    settings: {
      notifications: boolean,
      remindersTime: timestamp,
      privacy: {
        dataCollection: boolean,
        cloudSync: boolean
      }
    }
  }
}

// Pets Collection
pets: {
  userId: {
    name: string,
    stage: number,
    currentDay: number,
    lastInteraction: timestamp,
    needLevels: {
      love: number,
      expression: number,
      comfort: number
    },
    currentEmotion: string,
    reflectionStreak: number,
    milestones: [
      {
        day: number,
        completed: boolean,
        completedAt: timestamp,
        badge: string,
        title: string
      }
    ]
  }
}

// Reflections Collection
reflections: {
  userId: {
    reflectionId: {
      text: string,
      emotions: [string],
      date: timestamp,
      day: number,
      aiInsights: {  // Optional AI analysis of emotional patterns
        themes: [string],
        growthAreas: [string],
        recommendations: [string]
      }
    }
  }
}
```

### 4. Notification System

```javascript
// Daily Reminder Notifications
export const schedulePetReminderNotifications = async (user) => {
  // Cancel any existing notifications
  await Notifications.cancelAllScheduledNotificationsAsync();
  
  // Get user's preferred reminder time
  const reminderTime = user.settings.remindersTime || new Date();
  reminderTime.setHours(20); // Default to 8 PM if not set
  reminderTime.setMinutes(0);
  
  const messages = [
    "Your inner child pet is waiting to hear about your day",
    "How are you feeling today? Your pet wants to know",
    "Time for some emotional expression! Your pet misses you",
    "Your inner child needs nurturing. Reflect on today's emotions",
    "Don't forget to feed your inner child with today's feelings"
  ];
  
  // Schedule daily notification
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Check in with your Inner Child",
      body: messages[Math.floor(Math.random() * messages.length)],
      sound: true,
      priority: Notifications.AndroidNotificationPriority.HIGH,
      data: { screen: 'Pet' }
    },
    trigger: {
      hour: reminderTime.getHours(),
      minute: reminderTime.getMinutes(),
      repeats: true,
    },
  });
  
  // Schedule missed day notification (if no interaction for 24 hours)
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Your inner child is feeling neglected",
      body: "It's been a while since you've expressed your emotions. Take a moment to check in.",
      sound: true,
      priority: Notifications.AndroidNotificationPriority.HIGH,
      data: { screen: 'Pet' }
    },
    trigger: {
      seconds: 60 * 60 * 24, // 24 hours
      repeats: false,
    },
  });
};
```

### 5. Growth Milestone Celebrations

```javascript
function checkAndTriggerMilestones(petState) {
  const currentMilestone = petState.milestones.find(
    m => m.day <= petState.currentDay && !m.completed
  );
  
  if (currentMilestone) {
    // Mark milestone as completed
    dispatch({
      type: 'COMPLETE_MILESTONE',
      payload: {
        day: currentMilestone.day,
        completedAt: new Date().toISOString()
      }
    });
    
    // Show celebration modal
    dispatch({
      type: 'SHOW_CELEBRATION',
      payload: {
        title: `Milestone: ${currentMilestone.title}`,
        badge: currentMilestone.badge,
        message: getMilestoneMessage(currentMilestone.day),
        petStage: petState.stage
      }
    });
    
    // Trigger celebration animation and haptic feedback
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    
    // Send achievement notification
    Notifications.scheduleNotificationAsync({
      content: {
        title: "Evolution Milestone Achieved!",
        body: `${currentMilestone.title}: Your inner child is growing stronger.`,
        badge: 1,
        data: { screen: 'Pet' }
      },
      trigger: null // Send immediately
    });
  }
}

function getMilestoneMessage(day) {
  switch(day) {
    case 1:
      return "You've begun your journey of emotional expression. Your tiny kitten represents the start of healing your inner child.";
    case 10:
      return "Your consistent emotional expression is helping your inner child grow. Notice the first signs of healing.";
    case 20:
      return "Halfway through your journey! Your inner child is blossoming as you continue to acknowledge your feelings.";
    case 30:
      return "Your inner strength is emerging as you continue to nurture your emotional health.";
    case 41:
      return "Congratulations on completing your 41-day journey! Your inner child has evolved into a symbol of your emotional strength and resilience.";
    default:
      return "You've reached an important milestone in your journey!";
  }
}
```

## Key Mobile App Benefits

1. **Always Available**: Users can express emotions whenever they arise
2. **Private & Secure**: Journal entries are encrypted and protected
3. **Gentle Reminders**: Notifications help maintain the practice
4. **Visual Progress**: Pet evolution provides tangible representation of growth
5. **Offline Access**: Full functionality without internet connection
6. **Tactile Interaction**: Touch and haptic feedback create emotional connection with the pet

## Implementation Timeline

1. **Month 1**: Core functionality development
   - App architecture and navigation
   - Basic pet mechanics and animations
   - User authentication and profile creation
   
2. **Month 2**: Feature development
   - Reflection journal system
   - Pet growth algorithm refinement
   - Emotion tracking visualization
   
3. **Month 3**: Polish and testing
   - Animation and interaction refinement
   - User testing and feedback collection
   - Performance optimization
   
4. **Month 4**: Launch preparation
   - Beta testing with limited user group
   - Bug fixes and final adjustments
   - App store submission and marketing materials

## Next Steps

1. Develop wireframes and interactive prototypes using Figma
2. Finalize the emotional growth algorithm with input from psychology experts
3. Create custom pet animations for each growth stage
4. Develop secure, encrypted storage for emotion journal entries
5. Implement gamification elements to encourage consistent emotional expression

This mobile app will transform the pet concept into a powerful, always-available emotional companion that grows alongside the user throughout their 41-day evolution journey. 