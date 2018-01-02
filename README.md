# Simple_WEB_CALL_DEMO
https://github.com/joansrobbort
============================================================================================

source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target 'new_Project' do
    pod 'Alamofire', '~> 4.0’
    pod 'Kingfisher', '~> 3.0' 
    pod 'SVProgressHUD'
end

=======LOGAIN========>>>>>>>>>

import UIKit
import Alamofire
import SVProgressHUD
class LoginVC: UIViewController {
    @IBOutlet weak var txtUserName: UITextField!
    @IBOutlet weak var txtPAssword: UITextField!
    //MARK:- ViewController Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        //  SVProgressHUD.tintColorDidChange(UIColor.red)
    }
    func logainCall (){
        SVProgressHUD.show()
        Alamofire.request("http://bestvote.in/demo/index.php/User/login", method: .post, parameters: ["uname":txtUserName.text!,"password" :txtPAssword.text!],
                          encoding: URLEncoding.default,
                          headers: nil).responseJSON
            { (response:DataResponse<Any>) in
                
                switch(response.result) {
                case .success(_):
                    if let data = response.result.value{
                        //   print(data)
                        SVProgressHUD.dismiss()
                        let DicData = data as![String:AnyObject]
if (DicData["responce"] as? String) == "Fail"{
    let alert = UIAlertController(title: "Alert", message: DicData["message"] as? String, preferredStyle: UIAlertControllerStyle.alert)
        alert.addAction(UIAlertAction(title: "Ok", style: UIAlertActionStyle.default, handler: nil))
    self.present(alert, animated: true, completion: nil)
}
else{
let Storybord = UIStoryboard.init(name: "Main", bundle: nil)
let MovieViewController = Storybord.instantiateViewController(withIdentifier: "MovieVC")as! MovieVC
self.navigationController?.pushViewController(MovieViewController, animated:true )
                            
    }
}
break
    case .failure(_):
    SVProgressHUD.dismiss()
    print(response.result.error ?? "")
    break
                    
        }
    }
}
    //MARK:- TapToAction
    @IBAction func TapToLogin(_ sender: Any) {
        self.logainCall()
    }
}



=========LIStVIEW DETAILE===============>>>>>>>>>>>>>>>>>>>>>>>>>


import UIKit
import Alamofire
import SVProgressHUD
import Kingfisher
class MovieVC: UIViewController,UITableViewDelegate,UITableViewDataSource,UISearchBarDelegate{
    
    @IBOutlet weak var txtSearchbar: UISearchBar!
    @IBOutlet weak var tblMovieList: UITableView!
    
    var RefrenVIewController = UIRefreshControl()
    var arrrMovie = [MovieBean]()
    var ArrayMovieData = [MovieBean]()
    
    var filtered = [MovieBean]()
    var searchActive : Bool = false
    var multiFilter : Bool = true
    var CurrentDate = ""
    
    //MARK:- ViewController Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        self.MovieCall()
        txtSearchbar.delegate = self
        
    }
    
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        RefrenVIewController.addTarget(self, action: #selector(self.MovieCall), for: .valueChanged)
        self.RefrenVIewController.addSubview(tblMovieList)
        
        
    }
    //MARK:- SearchBar Delegate
    func searchBarTextDidBeginEditing(_ searchBar: UISearchBar) {
        searchActive = true
    }
    
    func searchBarTextDidEndEditing(_ searchBar: UISearchBar) {
        searchActive = false
    }
    
    func searchBarCancelButtonClicked(_ searchBar: UISearchBar) {
        searchActive = false
    }
    
    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        searchActive = false
    }
    
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        filtered.removeAll()
        // if multiFilter{
        filtered = ArrayMovieData.filter({ (Bean:MovieBean) -> Bool in
            let tmp: NSString = Bean.title as NSString
            let range = tmp.range(of: searchText, options: NSString.CompareOptions.caseInsensitive)
            return range.location != NSNotFound
        })
        
        if searchText.characters.count == 0{
            searchActive = false
        } else {
            searchActive = true
        }
        self.tblMovieList.reloadData()
    }
    
    func MovieCall(){
    SVProgressHUD.show()
    Alamofire.request("http://bestvote.in/demo/index.php/User/movie_list", method: .get, parameters:nil,encoding: URLEncoding.default,headers: nil).responseJSON
    { (response:DataResponse<Any>) in
    if self.RefrenVIewController.isRefreshing == false{
    SVProgressHUD.dismiss()
    }
    switch(response.result) {
    case .success(_):
    SVProgressHUD.dismiss()
    self.RefrenVIewController.endRefreshing()
    if let data = response.result.value{
    print(data)
    let DicData = data as? [String:Any]
    if let ArrayData = DicData!["data"] as? [Any]{
                            
    self.ArrayMovieData.removeAll()
    self.arrrMovie.removeAll()
                            
    for DataDic in ArrayData{
    if let ResultData = DataDic as? [String:Any]{
    let Bean = MovieBean()
                                    
    Bean.Id = (ResultData["id"] as? NSNumber as? Int)
    Bean.title = ResultData["title"] as Any as! String
    Bean.cover_image = ResultData["cover_image"] as Any as! String
    Bean.language = ResultData["language"] as Any as! String
    Bean.show_timing = ResultData["show_timing"] as Any as! String
    Bean.view = ResultData["view"] as Any as! String
    Bean.Description = ResultData["description"] as Any as! String
    Bean.release_date = ResultData["release_date"] as Any as! String
    Bean.ratings = ResultData["ratings"] as Any as! String
    self.multiFilter = true
    self.ArrayMovieData.append(Bean)
                                    
    }
}
   self.tblMovieList.reloadData()
        
}
        
    }
   break
                    
   case .failure(_):
   SVProgressHUD.dismiss()
   self.RefrenVIewController.endRefreshing()
   print(response.result.error ?? "")
   break
                    
        }
     }
}
    
    
//MARK:- TableView Delegate Metho
    
func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if(searchActive) {
            return filtered.count
        }
        if arrrMovie.count > 0 {
            return arrrMovie.count
        }
        return ArrayMovieData.count
    }
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell:MovieCell = tableView.dequeueReusableCell(withIdentifier: "Cell")as! MovieCell
        var Bean = MovieBean()
        if(searchActive){
            Bean = filtered[indexPath.row]
        }
        if arrrMovie.count > 0 {
            Bean = arrrMovie[indexPath.row]
        }
        else {
            Bean = ArrayMovieData[indexPath.row]
        }
        cell.lblTitle.text = Bean.title
        cell.lblview.text = Bean.Description
        cell.lblrealsedate.text = Bean.language
        let imgURl = URL(string: Bean.cover_image)
        cell.imgMovie.kf.setImage(with: imgURl)
        
        return cell
    }
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        
        var Bean = MovieBean()
        
        if arrrMovie.count > 0 {
            Bean = arrrMovie[indexPath.row]
        }
        else {
            Bean = ArrayMovieData[indexPath.row]
        }
        let Storybord = UIStoryboard.init(name: "Main", bundle: nil)
        let MovieDetailViewController = Storybord.instantiateViewController(withIdentifier: "MovieDetailVC")as! MovieDetailVC
        MovieDetailViewController.ResponceBean = Bean
        self.navigationController?.pushViewController(MovieDetailViewController, animated:true)
    }
    func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
        return true
    }
    
    internal func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        if (editingStyle == UITableViewCellEditingStyle.delete) {
            if ArrayMovieData.count > 0 {
                ArrayMovieData.remove(at: indexPath.row)
            }
            if arrrMovie.count > 0{
                arrrMovie.remove(at: indexPath.row)
            }
            
            tblMovieList.endUpdates()
            self.tblMovieList.reloadData()
        }
    }
    
@IBAction func TapToPickerView(_ sender: Any) {
    let optionMenu = UIAlertController(title: nil, message: "Choose Option Movie", preferredStyle: .actionSheet)
        
let GujaratiAction = UIAlertAction(title: "Gujarati", style: .default, handler:
    {
    (alert: UIAlertAction!) -> Void in
    self.arrrMovie .removeAll()
    for Bean in self.ArrayMovieData{
    if Bean.language == "Gujrati"{
    self.multiFilter = false
    self.arrrMovie.append(Bean)
    }
}
    self.tblMovieList.reloadData()
})
    let HindiAction = UIAlertAction(title: "Hindi", style: .default, handler:
{
    (alert: UIAlertAction!) -> Void in
    self.arrrMovie .removeAll()
    for Bean in self.ArrayMovieData{
    if Bean.language == "Hindi"{
    self.multiFilter = false
    self.arrrMovie.append(Bean)
    }
}
    self.tblMovieList.reloadData()
            
  })
let EnglishAction = UIAlertAction(title: "English", style: .default, handler:
    {
    (alert: UIAlertAction!) -> Void in
    self.arrrMovie .removeAll()
    for Bean in self.ArrayMovieData{
    if Bean.language == "English"{
    self.multiFilter = false
    self.arrrMovie.append(Bean)
    }
}
    self.tblMovieList.reloadData()
            
})
let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler:
{
            (alert: UIAlertAction!) -> Void in
    
})
    optionMenu.addAction(GujaratiAction)
    optionMenu.addAction(HindiAction)
    optionMenu.addAction(EnglishAction)
    optionMenu.addAction(cancelAction)
    self.present(optionMenu, animated: true, completion: nil)
    }

    
}
================================>>>>>>>>>>>>>>>>>>>>>>>>>>



class MovieDetailVC: UIViewController {

    @IBOutlet weak var lblLuaguage: UILabel!
    @IBOutlet weak var lblTitle: UILabel!
    @IBOutlet weak var lblDiscription: UILabel!
    @IBOutlet weak var lblRating: UILabel!
    @IBOutlet weak var lblReleaseDate: UILabel!
    @IBOutlet weak var lblShowTime: UILabel!
    @IBOutlet weak var MovieImg: UIImageView!
    @IBOutlet weak var lblView: UILabel!
    
    var ResponceBean  = MovieBean()
    
    //MARK:-ViewController life cycle
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        lblTitle.text = ResponceBean.title
        lblView.text = ResponceBean.view
        lblRating.text = (ResponceBean.ratings)
        
        lblLuaguage.text = ResponceBean.language
        lblShowTime.text = ResponceBean.show_timing
        
        lblReleaseDate.text = ResponceBean.release_date
        lblDiscription.text = ResponceBean.Description
       
        let ImgUrl = URL(string:ResponceBean.cover_image)
        self.MovieImg.kf.setImage(with: ImgUrl)
        
    }
    
    //MARK:- Button Back
    @IBAction func TapToBack(_ sender: Any) {
     self.navigationController?.popViewController(animated: true)
    }
}



































