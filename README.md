# php-script-ssl-checker-automatic-with-sslscan
System Requirments :
- Php : >7.4
- application sslscan
- terminal

1. Check if you have installed php
2. Install sslscan on ubuntu : ```bash sudo apt-get install sslscan```
3. Create 2 files ```bash domains.txt``` and ```bash check_ssl.php``` run code : touch ```bash domains.txt check_ssl.php```
4. on ```bash domains.txt``` input domain what you want make separate with new line, example :
```text
google.com
facebook.com
```
6.  on ```bash check_ssl.php``` copy paste this code
```php
<?php
// Read the list of domains from a file
$domain_file = 'domains.txt';
$domains = file($domain_file, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);

// Function to run sslscan and retrieve certificate information
function check_ssl($domain) {
    // Execute the sslscan command
    $command = "sslscan --no-colour $domain";
    echo "Checking SSL for: $domain\n"; // Displaying the process in the terminal
    $output = shell_exec($command);
    
    // Check for errors while executing the command
    if ($output === null) {
        return ['domain' => $domain, 'error' => 'Unable to run sslscan'];
    }
    
    // Extract Not valid before and Not valid after information
    preg_match('/Not valid before: (.*?)\n/', $output, $not_valid_before);
    preg_match('/Not valid after: (.*?)\n/', $output, $not_valid_after);
    
    // Calculate remaining days
    if (isset($not_valid_after[1])) {
        $expiry_date = strtotime(trim($not_valid_after[1]));
        $days_left = ($expiry_date - time()) / (60 * 60 * 24);
    } else {
        $days_left = 'Not found';
    }

    return [
        'domain' => $domain,
        'not_valid_before' => isset($not_valid_before[1]) ? trim($not_valid_before[1]) : 'Not found',
        'not_valid_after' => isset($not_valid_after[1]) ? trim($not_valid_after[1]) : 'Not found',
        'days_left' => $days_left
    ];
}

// Save results to a file
$result_file = 'ssl_results.txt';
file_put_contents($result_file, "Domain\tNot Valid Before\tNot Valid After\tDays Left\n", FILE_APPEND);

// Check SSL for each domain
foreach ($domains as $domain) {
    $result = check_ssl($domain);
    $line = "{$result['domain']}\t{$result['not_valid_before']}\t{$result['not_valid_after']}\t{$result['days_left']}\n";
    file_put_contents($result_file, $line, FILE_APPEND);
    echo "Result for $domain saved to $result_file.\n"; // Displaying status after saving results
}

echo "SSL check completed. Results saved in $result_file.\n";
?>
```
7.  to run this code, you can use terminal and run : ```bash php check_ssl.php```

thanks to : sslscan https://www.kali.org/tools/sslscan/
