# Converting to DNA & Go Modules
**General Instructions**

## Purpose
TODO

## Receipy

*do git commits along the way*

### dep -> go.mod

```
mkdir old
sudo chown -R garym:garym vendor/
mv vendor/ Gopkg.* old/
export GO111MODULE=on
go mod init bitbucket.org/reliefsoftware/service-user
```

copy `replaces`, `excludes` and `requires` from [existing go.mod eg  api-job](https://bitbucket.org/reliefsoftware/api-job/src/RS-1343-dna-mod/go.mod)


### go-blink -> go-blink/v2

replace all imports for 
`bitbucket.org/gotoblink/go-blink` and  `bitbucket.org/gotoblink/go-blink/xxx`
with 
`bitbucket.org/gotoblink/go-blink/v2` and  `bitbucket.org/gotoblink/go-blink/v2/xxx`

### move to dna

#### Replace
Regex find and replace will get most of them

```
"bitbucket.org/reliefsoftware/([^/]+)/proto/[^/]*"

"bitbucket.org/gotoblink/gb-godna/pb/$1"
```

```
"bitbucket.org/reliefsoftware/([^/]+)/proto/[^/]*/(.*)"

"bitbucket.org/gotoblink/gb-godna/pb/$1/$2"
```


#### Delete protos existing generated

find . -name "*.pb.go" | grep -v vendor | xargs rm 
find . -name "*.micro.go" | grep -v vendor  | xargs rm  
find proto -name "*.proto" | xargs rm

#### Fix validation and consts

```
$ go build
# bitbucket.org/reliefsoftware/service-user/handler
handler/account.go:118:49: undefined: service_user.TeamRoleAdmin
handler/account.go:131:54: undefined: service_user.TeamRoleAdmin
handler/account.go:179:17: undefined: constants.EventUserCreated
handler/account.go:186:35: undefined: constants.EventUserCreated
handler/account.go:397:16: undefined: constants.EventUserLogin
handler/account.go:412:12: req.Validate undefined (type *service_user.VerifyEmailRequest has no field or method Validate)
handler/account.go:460:16: undefined: constants.EventUserVerifyEmail
handler/account.go:543:16: undefined: constants.EventUserChangePassword
handler/common.go:29:2: undefined: constants.EventRunAllocated
handler/common.go:30:2: undefined: constants.EventUserInvitedToTeam
handler/account.go:543:16: too many errors
```

#####
add import 

eg
```
pb "bitbucket.org/gotoblink/gb-godna/pb/service-user"
```

##### move consts to go-blink/v2/contacts/XXXX

cp paste files, edit package names

use local dev ie replace in go.mod file

```
replace bitbucket.org/gotoblink/go-blink/v2 => ../../gotoblink/go-blink/v2
```


##### Wrap requests being validated

*Validator*
```
grep "Validate()" validate.go  | cut -d" " -f3 | cut -d")" -f1 | sed 's!\(.*\)!type \1 struct { *pb.\1 }!'
```
```
	pb "bitbucket.org/gotoblink/gb-godna/pb/service-user"
```

*Client*
import
```
	accountVal "bitbucket.org/reliefsoftware/service-user/proto/account"
```

wrap
```
	err := req.Validate()
->
	err := (accountVal.VerifyEmailRequest{req}).Validate()
```

For the text; copy go build output and do find replace

./([^.]*).*service_user.([^ ]*).*

err := ($1Val.$2{ req }).Validate()


