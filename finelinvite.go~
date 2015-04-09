package main

import (
	//"bytes"
	"code.google.com/p/gorest"
	"crypto/rand"
	"crypto/tls"
	"encoding/json"
	"fmt"

	"log"
	"net"
	"net/http"
	"net/mail"
	"net/smtp"
	//"log"

	"bytes"

	"io/ioutil"
)

type InvitEmail struct {
	SenderEmail   string
	ReceiverEmail string
}
type Token struct {
	Token string
}
type Activation struct {
	SenderEmail   string
	ReceiverEmail string
	Token         string
}

//Service Definition
type HelloService struct {
	gorest.RestService
	//gorest.RestService `root:"/tutorial/"`
	helloWorld     gorest.EndPoint `method:"GET" path:"/hello-world/" output:"string"`
	sayHello       gorest.EndPoint `method:"GET" path:"/hello/{name:string}" output:"string"`
	userRegister   gorest.EndPoint `method:"POST" path:"/UserRegister/" postdata:"InvitEmail"`
	userActivation gorest.EndPoint `method:"POST" path:"/UserActivation/" postdata:"Token"`
	userActive     gorest.EndPoint `method:"GET" path:"/Activate/{token:string}" output:"string"`
}

func main() {
	gorest.RegisterService(new(HelloService)) //Register our service
	http.Handle("/", gorest.Handle())
	http.ListenAndServe(":8787", nil)
}
func (serv HelloService) UserActive(token string) string {
	result := Objectstorecallingsearch(token)
	return result
}
func (serv HelloService) UserRegister(u InvitEmail) {

	fmt.Println(u.ReceiverEmail, "xxxxxxxx", u.SenderEmail)

	token := randToken()
	fmt.Println(token)
	invitationmail(u.SenderEmail, u.ReceiverEmail, token)
	serv.ResponseBuilder().SetResponseCode(200).Write([]byte("from :" + u.SenderEmail + "      to :" + u.ReceiverEmail))
}
func (serv HelloService) UserActivation(t Token) {
	Objectstorecallingsearch(t.Token)
	serv.ResponseBuilder().SetResponseCode(200)

}

func (serv HelloService) HelloWorld() string {
	return "Hello World"
}
func (serv HelloService) SayHello(name string) string {
	token := randToken()
	return "Hello " + name + token
}

//random token genarator
func randToken() string {
	b := make([]byte, 8)
	rand.Read(b)
	return fmt.Sprintf("%x", b)
}

func invitationmail(sender, receiver, token string) {
	from := mail.Address{"", sender}
	to := mail.Address{"", receiver}
	subj := "This is the email subject"
	body := "This is an example body.\n With two lines\n http://localhost:8787/Activate/" + token
	headers := make(map[string]string)
	headers["From"] = from.String()
	headers["To"] = to.String()
	headers["Subject"] = subj
	// Setup message
	message := ""
	for k, v := range headers {
		message += fmt.Sprintf("%s: %s\r\n", k, v)
	}
	message += "\r\n" + body
	// Connect to the SMTP Server
	servername := "smtp.gmail.com:465"
	host, _, _ := net.SplitHostPort(servername)
	auth := smtp.PlainAuth("", sender, PASSWORD, host)

	// TLS config
	//fmt.Print("7")
	tlsconfig := &tls.Config{
		InsecureSkipVerify: true,
		ServerName:         host,
	}

	// Here is the key, you need to call tls.Dial instead of smtp.Dial
	// for smtp servers running on 465 that require an ssl connection
	// from the very beginning (no starttls)
	//fmt.Print("8")
	conn, err := tls.Dial("tcp", servername, tlsconfig)
	if err != nil {
		log.Panic(err)
	}

	c, err := smtp.NewClient(conn, host)
	if err != nil {
		log.Panic(err)
	}

	// Auth
	if err = c.Auth(auth); err != nil {
		log.Panic(err)
	}

	// To && From
	if err = c.Mail(from.Address); err != nil {
		log.Panic(err)
	}

	if err = c.Rcpt(to.Address); err != nil {
		log.Panic(err)
	}

	// Data
	w, err := c.Data()
	if err != nil {
		log.Panic(err)
	}

	_, err = w.Write([]byte(message))
	if err != nil {
		log.Panic(err)
	}

	err = w.Close()
	if err != nil {
		log.Panic(err)
	} else {
		fmt.Println("\nMail sent sucessfully....")
	}

	c.Quit()
	var jsonStr = []byte("{\"Object\":{\"SenderEmail\":\"" + sender + "\",\"ReceiverEmail\":\"" + receiver + "\",\"Token\":\"" + token + "\",\"Activated\":\"False\"}, \"Parameters\":{\"KeyProperty\":\"ReceiverEmail\"}}")
	ObjectstorecallingInvite(jsonStr)

}

func Objectstorecallingsearch(token string) string {
	url := "http://duoworld.sossgrid.com:3000/com.duosoftware.com/newobject?keyword=" + token
	fmt.Println("URL:>", url)
	result := "false"
	//var jsonStr = []byte(`{"SenderEmail":"pamidu@duosoftware.com","ReceiverEmail":"ABC@live.com"}`)
	req, err := http.NewRequest("GET", url, nil)
	req.Header.Set("securityToken", "securityToken")
	req.Header.Set("log", "log")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	fmt.Println("response Status:", resp.Status)
	fmt.Println("response Headers:", resp.Header)
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println("response Body:", string(body))
	var jsonBlob = body
	var acti []Activation
	err1 := json.Unmarshal(jsonBlob, &acti)
	if err1 != nil {
		fmt.Println("error:", err1)
	}
	fmt.Printf("%+v", acti)
	for a := range acti {
		senderm := acti[a].SenderEmail
		revceivem := acti[a].ReceiverEmail
		tokenm := acti[a].Token
		if senderm != "" && revceivem != "" && tokenm != "" {
			var jsonStr = []byte("{\"Object\":{\"SenderEmail\":\"" + senderm + "\",\"ReceiverEmail\":\"" + revceivem + "\",\"Token\":\"" + tokenm + "\",\"Activated\":\"TRUE\"}, \"Parameters\":{\"KeyProperty\":\"ReceiverEmail\"}}")
			result = ObjectstorecallingInvite(jsonStr)
		}

	}
	return result

}
func ObjectstorecallingInvite(json []byte) string {
	url := "http://duoworld.sossgrid.com:3000/com.duosoftware.com/newobject"
	fmt.Println("URL:>", url)
	result := "false"
	//var jsonStr = []byte(`{"SenderEmail":"pamidu@duosoftware.com","ReceiverEmail":"ABC@live.com"}`)
	req, err := http.NewRequest("POST", url, bytes.NewBuffer(json))
	req.Header.Set("securityToken", "securityToken")
	req.Header.Set("log", "log")
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	} else {
		result = "true"
	}
	defer resp.Body.Close()

	fmt.Println("response Status:", resp.Status)
	fmt.Println("response Headers:", resp.Header)
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println("response Body:", string(body))
	return result

}
