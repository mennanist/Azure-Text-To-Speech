# Azure-Text-To-Speech
While implementing the Text-To-Speech service from Microsoft Azure, I was left quite confused, as there were different pages with different samples. so I'm explaining it all here for the future then providing the clean code in a python file:


azure text to speech PYTHON 3


first part: you need to import these packages. if they're not found use: pip install <package name>

    import os
    import requests
    import time
    from xml.etree import ElementTree

we'll create a class that we will later call on (we'll call it TextToSpeech)

    class TextToSpeech(object):
    
purpose of __init__ method is to initialize the values of instance members for the object. 
and subscription_key will later be defined when the class is called on in the code.
tts prompts the user to enter their own values (what they want to convert from text to speech) and timesstr records time to name your audio file later.

    def __init__(self, subscription_key):
        self.subscription_key = subscription_key
        self.tts = input("What would you like to convert to speech: ")
        self.timestr = time.strftime("%Y%m%d-%H%M")
        self.access_token = None
        
we get the token using the area we chose, for me it was: westus2.
list of regions can be found here: https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#rest-apis
you do not add subsciption key here because we're generalizing now and will later on add values.

    def get_token(self):
        fetch_token_url = "https://westus2.api.cognitive.microsoft.com/sts/v1.0/issuetoken"
        headers = {
        'Ocp-Apim-Subscription-Key': self.subscription_key
    }
        response = requests.post(fetch_token_url, headers=headers)
        self.access_token = str(response.text)
        
        
you get the base url from the list provided it's yourarea.tts.speech.microsoft.com/cognitiveservices/v1
and split that after the first dash and put the rest in the 'path' like done below.
here my area is: westus2.
you chose the output format for the audio file-- here mine is: audio-16khz-64kbitrate-mono-mp3
you ADD YOUR OWN USER AGENT, it's your resource name found on your azure portal.

    def save_audio(self):
        base_url = 'https://westus2.tts.speech.microsoft.com/'
        path = 'cognitiveservices/v1'
        constructed_url = base_url + path
        headers = {
        'Authorization': 'Bearer ' + self.access_token,
        'Content-Type': 'application/ssml+xml',
        'X-Microsoft-OutputFormat': 'audio-16khz-64kbitrate-mono-mp3',
        'User-Agent': 'YOUR USER AGENT'
        }
        
        
here we link to Speech Synthesis Markup Language (SSML).
and choose the language and version of the voice: https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/language-support
here i use ar-EG (arabic-egypt) but you need to put in your own language replacing every ar-EG with language and 'Hoda' with the version you choose from link above.

        xml_body = ElementTree.Element('speak', version='1.0')
        xml_body.set('{http://www.w3.org/XML/1998/namespace}lang', 'ar-EG')
        voice = ElementTree.SubElement(xml_body, 'voice')
        voice.set('{http://www.w3.org/XML/1998/namespace}lang', 'ar-EG')
        voice.set(
        'name', 'Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)')
        voice.text = self.tts
        body = ElementTree.tostring(xml_body)

        response = requests.post(constructed_url, headers=headers, data=body)
        

        
if response is 200: meaning if it's successful, write the audio file to the repository. else: let the user know something is wrong.
        
        if response.status_code == 200:
            with open('sample-' + self.timestr + '.wav', 'wb') as audio:
                audio.write(response.content)
                print("\nStatus code: " + str(response.status_code) +
                  "\nYour TTS is ready for playback.\n")
        else:
            print("\nStatus code: " + str(response.status_code) +
              "\nSomething went wrong. Check your subscription key and headers.\n")
class ready.


here you add YOUR OWN SUBSCRIPTION KEY FROM YOUR SERVICE ON AZURE PORTAL
            
    if __name__ == "__main__":
       subscription_key = "ADD KEY HERE"
       app = TextToSpeech(subscription_key)
       app.get_token()
       app.save_audio()
        


